


# Serialization

Java objects that you put into Hazelcast need to be serialized since Hazelcast is a distributed system. The data and its replicas are stored in different partitions on multiple nodes. The data you need may not be present on the local machine and Hazelcast retrieves that data from another machine. And this requires serialization.

Hazelcast serializes all your objects into an instance of `com.hazelcast.nio.serialization.Data`. `Data` is the binary representation of an object. 

Serialization is used when;

- key/value objects are added to a map,
- items are put in a queue/set/list,
- a runnable is sent using an executor service,
- an entry processing is performed within a map,
- an object is locked,and
- a message is sent to a topic


Hazelcast optimizes the serialization for the below types, and the user cannot override this behavior:

![image](images/OptimizedTypes.jpg)


Hazelcast also optimizes the following types. However, you can override them by creating a custom serializer and registering it. See [Custom Serialization](#custom-serialization) for more information.

![image](images/OptimizedTypesII.jpg)


As we have said, Hazelcast optimizes all of the above object types and you do not need to worry about their (de)serializations. 

When it comes to complex objects, below interfaces are used for serialization and deserialization.

- `java.io.Serializable`
- `java.io.Externalizable`

- `com.hazelcast.nio.serialization.DataSerializable`

- `com.hazelcast.nio.serialization.IdentifiedDataSerializable`

- `com.hazelcast.nio.serialization.Portable`




When Hazelcast serializes an object into `Data`:

**(1)** It first checks whether the object is an instance of `com.hazelcast.nio.serialization.DataSerializable`, 

**(2)** If above fails, then it checks if it is an instance of `com.hazelcast.nio.serialization.Portable`,

**(3)** If above fails, then it checks whether the object is of a well-known type like String, Long, Integer, etc. and user specified types like ByteArraySerializer or StreamSerializer,

**(4)** If above checks fail, Hazelcast will use Java serialization.

If all of the above checks do not work, then serialization will fail. When a class implements multiple interfaces, above steps are important to determine the serialization mechanism that Hazelcast will use. And when a class definition is required for any of these serializations, all the classes needed by the application should be on the classpath, Hazelcast does not download them automatically.

Let's dig into the details of the above serialization mechanisms in the following sections.

