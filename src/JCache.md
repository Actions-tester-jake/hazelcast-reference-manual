 
# Hazelcast JCache Implementation

***NOTE:*** *This feature will be included in the next Hazelcast release.* 

Starting with release 3.3.1, Hazelcast offers JCache API (JSR-107) implementation. JCache defines a standard Java API for caching temporary Java objects in memory. If retrieving or calculating data is costly, caching the objects in memory can help. By this way, you retrieve your data directly from the memory, which leads to improved performance. This chapter explains the usage of JCache with Hazelcast’s implementation.

For the full details of JCache, please visit its website on Java Community Process (JCP):

[https://www.jcp.org/en/jsr/detail?id=107](#https://www.jcp.org/en/jsr/detail?id=107)

## Setup

- Pull the latest from repo using the command `git pull origin jcache-preview` and use 
Maven install (or package) to build with `mvn clean install`.

- Add the file `hazelcast-3.3-JCACHE.jar` to your classpath.

- Download the `cache-api` from maven repo or add it as a dependency as shown below.

```xml
<dependency>
  <groupId>javax.cache</groupId>
  <artifactId>cache-api</artifactId>
  <version>1.0.0</version>
</dependency>
```

## Usage

After setting dependencies, you can start using JCache as described by the specification JSR 107.
Please note that Hazelcast specific configurations still can be performed as described by this document.

## Sample JCache Code

A sample code is shown below.

```java
CachingProvider cachingProvider = Caching.getCachingProvider();
CacheManager cacheManager = cachingProvider.getCacheManager();

//configure the cache
MutableConfiguration<String, String> config = new MutableConfiguration<String, String>();
config.setStoreByValue(true)
.setTypes(String.class, String.class)
.setExpiryPolicyFactory(AccessedExpiryPolicy.factoryOf(ONE_MINUTE))
.setStatisticsEnabled(false);

//create the cache
cacheManager.createCache(name, config);

//get the cache
Cache<String, Integer> cache = cacheManager.getCache(name, String.class, String.class);
cache.put("theKey", "Hello World");
String value = cache.get("theKey");
System.out.println(value);//prints 'Hello World'
```

## Code Samples

Please see [Hazelcast Code Samples](https://github.com/hazelcast/hazelcast-code-samples) for more examples.


## Work In Progress

Below features are still in progress and will be implemented in Hazelcast 3.3.1 release:

- Hazelcast Java client support for JCache
- Statistics and MXBean support 
- Synchronous listeners
- Near Cache extension
- Management Center integration
