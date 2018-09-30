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