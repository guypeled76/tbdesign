# Generic Local Cache

## Task:
You're tasked with writing a spec for a generic local cache with the following property: If the cache is asked for a key that it doesn't contain, it should fetch the data using an externally provided function that reads the data from another source (database or similar). What features do you think such a cache should offer? How, in general lines, would you implement it?

## Features:
1. ????How much do we need to cache? The amount is obviously limited by the fact that this is a local cache and not a distributed cache. How do we track the amount of memory we used? The most reasonable approach is to calculate memory size on each write and evict until we get to the configured amount of cache memory available. Another approach which might be simpler by prune to out of memory scenarios is to account for the amount of items only which will move the responsibility to the cache user. He will need to limit the maximum items based on a memory estimation of the item size range.
2. What should be the eviction policy if we need to free memory? We might add configuration for switching between eviction policies.
    * LRU (Least Recently Used) eviction policy nicely fits most of the use cases for caching. The idea is to evict entries that are not used as often based on the least recently used order. 
    * LFU (Least Frequently Used) eviction policy usually involve the system keeping track of the number of times an item is referenced. When the cache is full and requires more room the system will purge the items with the lowest reference frequency. 
    * FIFO eviction policy is based on First-In-First-Out (FIFO) algorithm which ensures that entry that has been in cache the longest will be evicted first. It is different from LruEvictionPolicy because it ignores the access order of entries. 
    * Random eviction policy which randomly chooses entries to evict. This eviction policy could be used for debugging and benchmarking purposes.
    * We could add an expiration time for each item to remove items even if the memory 
    limit was not achieved. That way we could free up items that had not been touched
    for a long time without waiting for memory to spike up.
3. How does eviction happen? We could have each write do the clean up task or we could have a background thread do the job at discrete intervals.
4. What are the data types that we should cache? Should we enforce serializable data types? Are we running in process or out of process. If we are running out of process this will not be optional and serializable types will be mandatory. 
5. The cache described in the task is a write through cache, where the cache is writing to the data source. In this task through the use of a delegate function or through the use of a callback interface to actually write the data to the data store. 
6. While a local write through cache solves the stale data potential of a single machine it does not account for multiple machines working with their own local cache. If we need consistency between machines we will still need to have a mechanism in place to communicate changes in sum kind of pub/sub mechanism. 
7. In terms of QPS (queries per seconds)/latency we might see the most bang for the bucks if we ensure that we are caching frequently used data. Because we are running locally we have minimum latency in terms of returning cached data but the limited amount of local memory might cause more frequent calls to uncached data which will in term produce a higher latency. 
8. Mutual exclusive locks should be acquired on a specific key bases to allow other key usages to continue while updating a given key. 

## Implementation Overview (in-process cache):

1. CacheStoreProvider - an interface or an abstract class that will be required to implement and provide and instance which will get and set values to the backend store.
2. CacheConfig - a class with definitions for cache behavior such as the eviction policy.
3. Cache - a class that will be the entry point for caching interaction.
    * Cache class API:
        * ctor(cacheStoreProvider, cacheConfig)
            * cacheStoreProvider - an instance of a class implementing or extending CacheStoreProvider.
            * cacheConfig - an instance of class CacheConfig defining the behavior of the cache.
        * get(key) - a method that given a key returns the value based on cached value or if it is not available using the CacheStoreProvider. If value is not cached we should lock based on a specific key in order to make sure that we only fetch the value from the store once.
        * set(key, value) - a method that given a key and value will update the cache after storing the value in the backend storage using the CacheStoreProvider. While updating we should acquire a lock for the key to ensure that cached data and data in the store are in sync.
    * Cache class implementation:
        * The cache class should hold a single member such as the LRUMap which will be a map that tracks it's entries by order, frequency, expiration and provided a call to a cleanup method will evict the relevant members. The member type will be based on the configuration provided in the construction of the Cache class.



## Implementation References (java):
1. [LRUMap](https://commons.apache.org/proper/commons-collections/apidocs/org/apache/commons/collections4/map/LRUMap.html) - 
A Map implementation with a fixed maximum size which removes the least recently used entry if an entry is added when full. 

