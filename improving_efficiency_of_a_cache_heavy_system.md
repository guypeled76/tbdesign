# Improving the efficiency of a cache heavy system

## Task:
You are tasked with improving the efficiency of a cache heavy system, which has the following
properties/architecture:

The system has 2 components, a single instance backend and several frontend instances.

1. The backend generates data and writes it to a relational database that is replicated to
multiple data centers.

2. The frontends handle client requests (common web traffic based) by reading data from
the database and serving it. Data is stored in a local cache for an hour before it expires
and has to be retrieved again. The cache’s eviction policy is LRU based.

There are two issues with the implementation above:

1. It turns out that many of the database accesses are redundant because the underlying
data didn't actually change.
2. On the other hand, a change isn't reflected until the cache TTL elapses, causing
staleness issues.

Offer a solution that fixes both of these problems. How would the solution change if the data
was stored in Cassandra and not a classic database?

## Solution

With the current cache architecture, cache items do not update when the data on the database updates.  That is way we have stale data and we are relaying on TTL to update the cache items and those having redundant queries to the data base.

This is really a pooling architecture which the cache layer is the client and the database as the server. If we lower the TTL time we will have less time for stale data but more redundant queries to database. If we increase the TTL we will have less redundant queries to the database but more time of stale data.

We need to introduce a pub/sub system in to the architecture to allow publishing of data changes to the cache layer instances. In the relational database architecture this is usually done by using triggers that publish change messages to the pub/sub system. This approach also works across data centers cause we can have triggers in the replication of databases which will notify the other data center cache layer the same way it had been done in the data center that was updated.

The messages sent to the pub/sub system should be rather simple as ideally it would contain a key or list of keys that should be invalidated. The subscribers of the pub/sub system will be the different cache layer instances which will now get notified when they need to delete stale data.

LRU and TTL eviction policies will still be used to manage the memory consumption but now we are not relaying on TTL to account for data updates. The time it will take to remove stale data will be dependent on the time it takes to receive a message from the backend data updates. While the consistency model will still be an eventually consistent model, the updating time will be more predictable.

The pub/sub approach for invalidating the stale data will solve the redundant database queries we had in the previous architecture as we now have a way to actually know if data has changed and don't need to pull data again to see if there was a change.

The real complexity with the new architecture of using a pub/sub system as a way to invalidate stale data in the cache layer is identifying which cache keys should we update. Introducing casandra as the backend data store allows us to cache the entries based on the casandra model which could be described as a key value storage which the key is a combination of table name table primary key and value as the actual record. 

The previous paragraph is obviously a simplification of caching needs cause we might need to store a list of internal record list stored in casandra such as users friends list but we can still use the table name and primary key to generate a predictable key. That way if we update the friends list with in a user record we could publish a change message to the pub/sub system that states a a specific change with a key like this '[table]_[primarykey]_friends'. 

## Related Resources:
1. [Facebook-Memcached](https://www.youtube.com/watch?v=UH7wkvcf0ys) - Mark Zuckerberg talks about how the company uses memcached for caching and storage performance. Facebook uses MySql databases and memcache and invalidates cache using triggers and a pub/sub approach.
