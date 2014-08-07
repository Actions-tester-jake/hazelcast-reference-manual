

## Default Hazelcast.xml Wrap Up

In Hazelcast, below configuration elements are available.

- group
- management-center
- network
- partition-group
- executor-service
- queue
- map
- multimap
- list
- set
- jobtracker
- semaphore
- serialization
- services



### `group` Tag

This configuration is for ???. It has below sub-tags.

- name:
- password:


### `management-center` Tag

This configuration is for ???. It only has the attribute `enabled`.

### `network` Tag

This configuration is for ???. It has below sub-tags.

- port:
- outbound-ports:
- join:
- interfaces:
- ssl:
- socket-interceptor:
- symmetric-encryption:
- public-address:

### `partition-group` Tag

This configuration is for ???. It only has the attribute `enabled`.

### `executor-service` Tag

This configuration is for ???. It has below attributes.

- pool-size:
- queue-capacity:


### `queue` Tag

This configuration is for ???. It has below attributes.

- max-size:
- backup-count:
- async-backup-count:
- empty-queue-ttl:
- item-listeners:
- queue-store:
- statistics-enabled:

### `map` Tag

This configuration is for ???. It has below attributes.

- in-memory-format:
- backup-count:
- async-backup-count:
- read-backup-data:
- time-to-live-seconds:
- max-idle-seconds:
- eviction-policy:
- max-size:
- eviction-percentage:
- merge-policy:
- statistics-enabled:
- map-store:
- near-cache:
- wan-replication-ref:
- indexes:
- entry-listeners:
- partition-strategy:


### `multimap` Tag

This configuration is for ???. It has below attributes.

- backup-count:
- async-backup-count:
- statistics-enabled:
- value-collection-type:
- entry-listeners:
- partition-strategy:


### `topic` Tag

This configuration is for ???. It has below attributes.

- statistics-enabled:
- global-ordering-enabled:
- message-listeners:


### `list` Tag

This configuration is for ???. It has below attributes.

- backup-count:
- async-backup-count:
- statistics-enabled:
- max-size:
- item-listeners:
- statistics-enabled:

### `set` Tag

This configuration is for ???. It has below attributes.

- backup-count:
- async-backup-count:
- statistics-enabled:
- max-size:
- item-listeners:
- statistics-enabled:




### `jobtracker` Tag

This configuration is for ???. It has below attributes.

- max-thread-size:
- queue-size:
- retry-count:
- chunk-size:
- communicate-stats:
- topology-changed-strategy:


### `semaphore` Tag

This configuration is for ???. It has below attributes.

- initial-permits:
- backup-count:
- async-backup-count:


### `serialization` Tag

This configuration is for ???. It has below attributes.

- portable-version:

### `services` Tag

This configuration is for ???. It only has the attribute `enabled`.












Below is the `hazelcast.xml` configuration file that comes with the release, located at `bin` folder.

```xml
<hazelcast xsi:schemaLocation="http://www.hazelcast.com/schema/config hazelcast-config-3.3.xsd"
           xmlns="http://www.hazelcast.com/schema/config"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <group>
        <name>dev</name>
        <password>dev-pass</password>
    </group>
    <management-center enabled="false">http://localhost:8080/mancenter</management-center>    
    <network>
        <port auto-increment="true" port-count="100">5701</port>
        <outbound-ports>
            <ports>0</ports>
        </outbound-ports>
        <join>
            <multicast enabled="true">
                <multicast-group>224.2.2.3</multicast-group>
                <multicast-port>54327</multicast-port>
            </multicast>
            <tcp-ip enabled="false">
                <interface>127.0.0.1</interface>
            </tcp-ip>
            <aws enabled="false">
                <access-key>my-access-key</access-key>
                <secret-key>my-secret-key</secret-key>
                <region>us-west-1</region>
                <host-header>ec2.amazonaws.com</host-header>
                <security-group-name>hazelcast-sg</security-group-name>
                <tag-key>type</tag-key>
                <tag-value>hz-nodes</tag-value>
            </aws>
        </join>
        <interfaces enabled="false">
            <interface>10.10.1.*</interface>
        </interfaces>
        <ssl enabled="false" />
        <socket-interceptor enabled="false" />
        <symmetric-encryption enabled="false">
            <algorithm>PBEWithMD5AndDES</algorithm>
            <salt>thesalt</salt>
            <password>thepass</password>
            <iteration-count>19</iteration-count>
        </symmetric-encryption>
    </network>   
    <partition-group enabled="false"/>   
    <executor-service name="default">
        <pool-size>16</pool-size>
        <queue-capacity>0</queue-capacity>
    </executor-service>   
    <queue name="default">
        <max-size>0</max-size>
        <backup-count>1</backup-count>
        <async-backup-count>0</async-backup-count>
        <empty-queue-ttl>-1</empty-queue-ttl>
    </queue>   
    <map name="default">
        <in-memory-format>BINARY</in-memory-format>
        <backup-count>1</backup-count>
        <async-backup-count>0</async-backup-count>
        <time-to-live-seconds>0</time-to-live-seconds>
        <max-idle-seconds>0</max-idle-seconds>
        <eviction-policy>NONE</eviction-policy>
        <max-size policy="PER_NODE">0</max-size>
        <eviction-percentage>25</eviction-percentage>
        <merge-policy>com.hazelcast.map.merge.PassThroughMergePolicy</merge-policy>
    </map>
    <multimap name="default">
        <backup-count>1</backup-count>
        <value-collection-type>SET</value-collection-type>
    </multimap>
    <multimap name="default">
        <backup-count>1</backup-count>
        <value-collection-type>SET</value-collection-type>
    </multimap>
    <list name="default">
        <backup-count>1</backup-count>
    </list>
    <set name="default">
        <backup-count>1</backup-count>
    </set>
    <jobtracker name="default">
        <max-thread-size>0</max-thread-size>
        <queue-size>0</queue-size>
        <retry-count>0</retry-count>
        <chunk-size>1000</chunk-size>
        <communicate-stats>true</communicate-stats>
        <topology-changed-strategy>CANCEL_RUNNING_OPERATION</topology-changed-strategy>
    </jobtracker>
    <semaphore name="default">
        <initial-permits>0</initial-permits>
        <backup-count>1</backup-count>
        <async-backup-count>0</async-backup-count>
    </semaphore>
    <serialization>
        <portable-version>0</portable-version>
    </serialization>
    <services enable-defaults="true" />
</hazelcast>
```

<br></br>