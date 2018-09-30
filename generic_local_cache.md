# Generic Local Cache

## Task:
You're tasked with writing a spec for a generic local cache with the following property: If the cache is asked for a key that it doesn't contain, it should fetch the data using an externally provided function that reads the data from another source (database or similar). What features do you think such a cache should offer? How, in general lines, would you implement it?

## Features:
1. How much do we need to cache? The amount is obviously limited by the fact that this is a local cache and not a distributed cache. How do we track the amount of memory we used?
2. What should be the eviction policy if we need to free memory? We might add configuration for switching between eviction policies.
    * LRU (Least Recently Used) eviction policy nicely fits most of the use cases for caching. The idea is to evict entries that are not used as often based on the least recently used order. 
    * FIFO eviction policy is based on First-In-First-Out (FIFO) algorithm which ensures that entry that has been in cache the longest will be evicted first. It is different from LruEvictionPolicy because it ignores the access order of entries. 
    * Random eviction policy which randomly chooses entries to evict. This eviction policy is mainly used for debugging and benchmarking purposes.
3. What are the data types that we should cache? Should we enforce serializable data types? Are we running in process or output of process. If we are running out of process this might will not be optional and serializable types will be mandatory. 
4. The cache described in the task is a write through cache, where the cache is writing to the data source. In this task through the use of a delegate function or through the use of a callback interface to actually write the data to the data store. 
5. While a local write through cache solves the stale data potential of a single machine it does not account for multiple machines working with their own local cache. If we need consistency between machines we will still need to have a mechanism in place to communicate changes in sum kind of pub/sub mechanism.
6. In terms of QPS (queries per seconds)/latency we might see the most bang for the bucks if we ensure that we are caching frequently used data. Because we are running locally we have minimum latency in terms of returning cached data but the limited amount of local memory might cause more frequent calls to uncached data which will in term produce a higher latency. 

