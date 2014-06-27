
## Aggregators

Based on the Hazelcast MapReduce framework Aggregators are ready-to-use data aggregations. Those are typical operations like
sum up values, finding minimum or maximum values, calculating averages and other operations as you expect them to be available in
the relational database world.  

Aggregation operations are implemented, as mentioned above, on top of the MapReduce framework and therefor all operations can be
achieved using pure map-reduce calls but using the Aggregation feature is more convenient for a big set of standard operations.

### Aggregations Basics

This short chapter will quickly guide you through the basics of the Aggregations framework and the available classes. We also
will implement a first base example.

Aggregations are available on both `com.hazelcast.core.IMap` and `com.hazelcast.core.MultiMap` interfaces using the `aggregate`
methods. Two possible overloads of that method are available to support customized resource management of the underlying
MapReduce framework by supplying a custom configured `com.hazelcast.mapreduce.JobTracker` instance. On how to configure
the MapReduce framework please see the section about [JobTracker Configuration]. We will later see another way to configure the
automatically used MapReduce framework if no special `JobTracker` is supplied.

To make Aggregations most convenient usable and future proof, the API is already heavily optimized for Java 8 and future version
but still fully compatible to any Java version Hazelcast supports (Java 6 and Java 7). The biggest difference is how you have to
work with the Java generics since on Java 6 and 7 the generics resolving process is not as strong as on Java 8 and upcoming
versions. In addition to that the whole Aggregations API has full Java 8 Project Lambda (or Closure, 
[JSR 335](https://jcp.org/en/jsr/detail?id=335)) support.

For illustration of the differences in Java 6 and 7 in comparison to Java 8 we now will have a quick look at both sourcecode
examples. After that the documentation will go on using Java 8 syntax to keep examples short and easy to understand.

The first basic example will produce the sum of some int values stored in a Hazelcast IMap. This is a very basic example not yet
using a lot of the functionality of the Aggregations framework but will already perfectly show the main difference.

```java
IMap<String, Integer> personAgeMapping = hazelcastInstance.getMap( "person-age" );
for (int i = 0; i < 1000; i++) {
  String lastName = RandomUtil.randomLastName();
  int age = RandomUtil.randomAgeBetween( 20, 80 );
  personAgeMapping.put( lastName, Integer.valueOf( age ) );
}
```

Since our demo data are now prepared we can have a look at how to produce the sums in the different Java versions.

#### Aggregations and Java 6 or Java 7

Since Java 6 and 7, as mentioned earlier, are not as strong on resolving generics as Java 8 we need to be a bit more verbose
on what we write or you might want to consider using raw types and breaking the type safety to ease this process.

For a short introduction on what the following lines mean have a quick look at the sourcecode comments. We will deeper dig into
the different options in a bit. 

```java
// No filter applied, select all entries
Supplier<String, Integer, Integer> supplier = Supplier.all();
// Choose the sum aggregation
Aggregation<String, Integer, Integer> aggregation = Aggregations.integerSum();
// Execute the aggregation
int sum = personAgeMapping.aggregate( supplier, aggregation );
```

#### Aggregations and Java 8

On Java 8 the Aggregations API looks much simpler since Java is now able to resolve our generic parameters for us. That means
the above lines of source will end up in one line on Java 8.

```
int sum = personAgeMapping.aggregate( Supplier.all(), Aggregations.integerSum() );
```

As you can see this really looks stunning and easy to use.

#### Quick look at the MapReduce Framework

As mentioned before the Aggregations implementation is based on the Hazelcast MapReduce framework and therefor you might find
overlaps of the different APIs and we've already seen one before. One overload of the `aggregate` method can be supplied with
a `JobTracker` which is part of the MapReduce framework.

If you are going to implement your own aggregations you also end up implementing them using a mixture of the Aggregations and
the MapReduce API. If you are looking forward to implement your own aggregation, e.g. to make the life of colleagues easier,
please read [Implementing Aggregations].

For the full MapReduce documentation please see [MapReduce].


### Introduction to Aggregations API

Following the basic example we now want to look into the real API, in the possible options and what can be achieved using the
Aggregations API. To work on some more and deeper examples let's quickly have a look at the available classes and interfaces and
discuss their usage.

#### Supplier

The `com.hazelcast.mapreduce.aggregation.Supplier` is used to provide filtering and data extraction to the aggregation operation.
This class already provides a few different static methods to achieve most common cases. We already learned about `Supplier.all()`
which accepts all incoming values and does not apply and data extraction or transformation upon them before supplying them to
the aggregation function itself.

For filtering data sets you, by default, have two different options. You can either supply a `com.hazelcast.query.Predicate`
if you want to filter on values and / or keys or a `com.hazelcast.mapreduce.KeyPredicate` if you can decide directly on the data
key without the need to deserialize the value yet.

As mentioned above all APIs are fully Java 8 and Lambda compatible so let's have a look on how we can do basic filtering using
those two options.

First we have a look at a `KeyPredicate` and only accept people whose last name is Jones.

```java
Supplier<...> supplier = Supplier.fromKeyPredicate(
    lastName -> "Jones".equalsIgnoreCase( lastName )
);
```

Using the standard Hazelcst `Predicate` interface you can also filter based on the value of an data entry. For example you can
only select values which are divisible without remainder by 4 using the following example. 

```java
Supplier<...> supplier = Supplier.fromPredicate(
    entry -> entry.getValue() % 4 == 0
);
```

Beside from the fact that a `Supplier` is used for filtering it is also used to extract or transform data before supplying them
to the aggregation operation itself. We will now look into a short example on how to transform the input value to a string.
 
```java
Supplier<String, Integer, String> supplier = Supplier.all(
    value -> Integer.toString(value)
);
```

Apart from the fact we transformed the input value of type int (or Integer) to a string we can see, that the generic information
of the resulting `Supplier` has changed as well, indicating that we now would have an aggregation working on string values.

The last bit to say about the `Supplier` is that it is possible to chain multiple filtering rules so let's combine all of the
above examples into one rule set:

```java
Supplier<String, Integer, String> supplier =
    Supplier.fromKeyPredicate(
        lastName -> "Jones".equalsIgnoreCase( lastName ),
        Supplier.fromPredicate(
            entry -> entry.getValue() % 4 == 0,  
            Supplier.all( value -> Integer.toString(value) )
        )
    );
```

#### Aggregation and Aggregations

The `com.hazelcast.mapreduce.aggregation.Aggregation` interface defines the aggregation operation itself. It contains a set of
MapReduce API implementations like `Mapper`, `Combiner`, `Reducer`, `Collator`. These implementations are normally unique to
the chosen `Aggregation`. This interface can also be implemented with your aggregation operations based on map-reduce calls. To
find deeper information on this please have a look at [Implementing Aggregations].

A common predefined set of aggregations are provided by the `com.hazelcast.mapreduce.aggregation.Aggregations` class. This class
contains typesafe aggregations of the following types:

 - Average (Integer, Long, Double, BigInteger, BigDecimal)
 - Sum (Integer, Long, Double, BigInteger, BigDecimal)
 - Min (Integer, Long, Double, BigInteger, BigDecimal, Comparable)
 - Max (Integer, Long, Double, BigInteger, BigDecimal, Comparable)
 - DistinctValues
 - Count

#### PropertyExtractor

We already used the `com.hazelcast.mapreduce.aggregation.PropertyExtractor` interface before when we had a look at the example
on how to use a `Supplier` to transform a value to another type. It can also be used to extract attributes from values what we
will look at now.

```java
class Person {
  private String firstName;
  private String lastName;
  private int age;
  
  // getters and setters
}

PropertyExtractor<Person, Integer> propertyExtractor = (person) -> person.getAge();
```

In this example we extract the value from the person's age attribute and so the value type changes from person to Integer which
is again reflected in the generics information to stay typesafe.

`PropertyExtractor`s are meant to be used for any kind of transformation of data, you might even want to have multiple
transformation steps chained one after another.

#### Aggregation Configuration

As stated before the easiest way to configure the resources used by the underlying MapReduce framework is to supply a `JobTracker`
to the aggregation call itself by passing it to either `IMap::aggregate` or `MultiMap::aggregate`.

There is also a second way on how to implicitly configure the underlying used `JobTracker`. If no specific `JobTracker` was
passed for the aggregation call an internally used one will be created using a naming specification as the following:

For `IMap` aggregation calls the naming spec is created as:
`hz::aggregation-map-` and concatenated the name of the map

For `MultiMap` it is very similar:
`hz::aggregation-multimap-` and concatenated the name of the multimap

Knowing that we can configure the `JobTracker` in the Hazelcast configuration file as expected and as name we use the previously
built name due to the specification. For more information on configuration of the `JobTracker` please see
[JobTracker Configuration]. 

To finish this section let's have a quick example for the above naming specs:

```java
IMap<String, Integer> map = hazelcastInstance.getMap( "mymap" );

// The internal JobTracker name resolves to 'hz::aggregation-map-mymap' 
map.aggregate(...);
```

```java
MultiMap<String, Integer> multimap = hazelcastInstance.getMultiMap( "mymultimap" );

// The internal JobTracker name resolves to 'hz::aggregation-multimap-mymultimap' 
multimap.aggregate(...);
```


### Implementing Aggregations
