## Threading Model

Your application server has its own thread. Hazelcast does not use these - it manages its own threads.

### IO Threading

Hazelcast uses a pool of thread performng (n)IO; so there is not a single thread doing all the IO, but there are multiple.
The number of IO threads can be configured using `hazelcast.io.thread.count` system property and defaults to 3. 

IO threads wait for the `Selector.select` to complete. When sufficient bytes for a Packet have been received,
the Packet object is created. This Packet is then sent to the System where it is de-multiplexed. If the Packet header
signals that it is an operation/response, it is handed over to the operation service (please see [Operation Threading](#operation-threading)). If the Packet 
is an event, it is handed over to the event service (please see [Event Threading](#event-threading)). 

### Event Threading

Hazelcast uses a shared event system to deal with components that rely on events like topic, collections listeners and near-cache. 

Each cluster member has an array of event threads and each thread has its own work queue. When an event is produced,
either locally or remote, an event thread is selected (depending on if there is a message ordering) and the event is placed
in the work queue for that event thread.

The following properties
can be set to alter the behavior of the system:

* `hazelcast.event.thread.count`: Number of event-threads in this array. Its default value is 5.
* `hazelcast.event.queue.capacity`: Capacity of the work queue. Its default value is 1000000.
* `hazelcast.event.queue.timeout.millis`: Timeout for placing an item on the work queue. Its default value is 250.

If you process a lot of events and have many cores, changing the value of `hazelcast.event.thread.count` property to
a higher value is a good idea. By this way, more events can be processed in parallel.

Multiple components share the same event queues. If there are 2 topics, say A and B, for certain messages
they may share the same queue(s) and hence event thread. If there are a lot of pending messages produced by A, then B needs to wait.
Also, when processing a message from A takes a lot of time and the event thread is used for that, B will suffer from this. 
That is why it is better to offload processing to a dedicate thread (pool) so that systems are better isolated.

If events are produced at a higher rate than they are consumed, the queue will grow in size. To prevent overloading system
and running into an `OutOfMemoryException`, the queue is given a capacity of 1M million items. When the maximum capacity is reached, the items are
dropped. This means that the event system is a 'best effort' system. There is no guarantee that you are going to get an
event. It can also be that Topic A has a lot of pending messages, and therefore B cannot receive messages because the queue
has no capacity and messages for B are dropped.

### IExecutor Threading:

Executor threading is straight forward. When a task is received to be executed on Executor E, then E will have its
own `ThreadPoolExecutor` instance and the work is put on the work queue of this executor. So, Executors are fully isolated, but of course, they will share the same underlying hardware; most importantly the CPUs. 

The IExecutor can be configured using the `ExecutorConfig` (programmatic configuration) or using `<executor>` (declarative configuration).

### Operation Threading:

There are 2 types of operations:

* Operations that are aware of a certain partition, e.g. `map.get`
* Operations that are not partition aware, e.g. `map.size`

Each of these types has a different threading model explained below.

#### Partition-aware Operations

To execute partition-aware operations, an array of operation threads is created. Size of this array is by default two times the number of cores. It can be changed using the `hazelcast.operation.thread.count` property.

Each thread has its own work queue and it will consume messages from this work queue. If a partition-aware operation needs
to be scheduled, the right thread is found using the below formula:

`threadIndex = partitionId % partition-thread-count`

And then the operation is put in the work queue of that worker.

This means that a single operation thread can execute operations for multiple partitions; if there are 271 partitions and
10 partition-threads, then roughly every operation thread will execute operations for 10 partitions.

If one partition is slow, e.g. a map EntryProcessor running slow user code, it will block the execution of other partitions
that map to the same operation thread. So it could be that partition 1 and partition 26 of the same map use the same operation
thread because they map to the same partition. It can even be that partition 1 for map X and partition 26 of map Y, which
are completely independent data structures, still contend for the same thread. So you need to be careful about starving
a partition of threading resources.

Currently, there is no support for work stealing; so different partitions, that map to
the same thread may need to wait till one of the partitions is finished, even though there are other free partition-operation threads
available.

<font color='red'>**Example**</font>

Take a 3 node cluster. Two members will have 90 primary partitions and one member will have 91 primary partitions. Let's
say you have one CPU and 4 cores per CPU. By default, 8 operation threads will be allocated to serve 90 or 91 partitions.


#### Non Partition-aware Operations

To execute non partition-aware operations, e.g. `map.size`, generic operation threads are used. When the Hazelcast instance is started,
an array of operation threads is created. Size of this array also is by default two times the number of cores. It can be changed using the
`hazelcast.operation.generic.thread.count` property.

Unlike the partition-aware operation threads, all the generic operation threads share the same work queue: `genericWorkQueue`.

If a non partition-aware operation needs to be executed, it is placed in that work queue and any generic operation thread can execute it.
The big advantage is that you automatically have work balancing since any generic operation thread is allowed to pick up work from 
this queue.

The disadvantage is that this shared queue can be a point of contention; however, practically we do not see this as in production
as performance is dominated by I/O and the system is not executing that many non partition-aware operations.
 
#### Priority Operations
 
In some cases the system needs to execute operations with a higher priority, e.g. an important system operation. To support priority
operations we do the following:

* For partition-aware operations: each partition thread has of course its own work queue. But apart from that, it also has a priority
  work queue. It will always check this priority queue, before it is going to process work from its normal work queue.

* For non partition-aware operations: next to the `genericWorkQueue`, there also is a `genericPriorityWorkQueue`. So when a priority operation
 needs to be executed, it is put in this `genericPriorityWorkQueue`. And just like the partition-aware operation threads, a generic
 operation thread, will first check the `genericPriorityWorkQueue` for work. 
 
Because a worker thread will block on the normal work queue (either partition specific or generic) it could be that a priority operation
is not picked up because it will not be put on the queue it is blocking on. We always send a 'kick the worker' operation that does 
nothing else than trigger the worker to wakeup and check the priority queue. 

#### Operation-response and Invocation-future

When an Operation is invoked, a `Future` is returned. Let's take the below sample code. 

```java
GetOperation op = new GetOperation(mapName, key)
Future f = operationService.invoke(op)
f.get)
```

So, the calling side blocks for a reply. In this case, `GetOperation` is set in the work queue for the partition of `key`, where
it eventually is executed. On execution, a response is returned and placed on the `genericWorkQueue` where it is executed by a 
"generic operation thread". This thread will signal the `future` and notifies the blocked thread that a response is available. 
In the future we will expose this Future to the outside world, and we will provide the ability to register a completion listener 
so you can do asynchronous calls. 

#### Local Calls

When a local partition-aware call is done, an operation is made and handed over to the work queue of the correct partition operation thread
and a future is returned. When the calling thread calls get on that future, it will acquire a lock and wait for the result 
to become available. When a response is calculated, the future is looked up, and the waiting thread is notified.  

In the future, this will be optimized to reduce the amount of expensive systems calls like `lock.acquire`/`notify` and the expensive
interaction with the operation-queue. Probably, we will add support for a caller-runs mode, so that an operation is directly executed on
the calling thread.