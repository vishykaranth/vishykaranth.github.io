## Caching
_"There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton_

![caching](/contents/2023/system-design/images/system-design/chapter-I/caching/caching.png)

- A cache's primary purpose is to increase data retrieval performance by reducing the need to access the underlying slower storage layer. 
  - Trading off capacity for speed, 
  - a cache typically stores a subset of data transiently, 
  - in contrast to databases whose data is usually complete and durable.
- Caches take advantage of the locality of reference principle _"recently requested data is likely to be requested again"._
- Load balancing helps you scale horizontally across an ever-increasing number of servers,
    - but caching will enable you to make vastly better use of the resources you already have,
    - as well as making otherwise unattainable product requirements feasible.
- Caches take advantage of the locality of reference principle:
    - recently requested data is likely to be requested again.
- They are used in almost every layer of computing: hardware, operating systems, web browsers, web applications and more.
- A cache is like short-term memory: it has a limited amount of space, but is typically faster than the original data source and contains the most recently accessed items.
- Caches can exist at all levels in architecture but are often found at the level nearest to the front end, where they are implemented to return data quickly without taxing downstream levels.
- How to design a cache system?
    - Cache system is a widely adopted technique in almost every applications today.
    - In addition, it applies to every layer of the technology stack.
    - For instance, at network area cache is used in DNS lookup and in web server cache is used for frequent requests.
    - A cache system stores common used resources (maybe in memory) and when next time someone requests the same resource, the system can return immediately.
    - Cache increases the system efficiency by consuming more storage space.



## Caching and Memory

- Like a computer's memory, 
  - a cache is a compact, fast-performing memory that stores data in a hierarchy of levels, 
  - starting at level one, and progressing from there sequentially. 
  - They are labeled as L1, L2, L3, and so on. 
  - A cache also gets written if requested, 
    - such as when there has been an update and 
    - new content needs to be saved to the cache, 
    - replacing the older content that was saved.
- No matter whether the cache is read or written, 
  - it's done one block at a time. 
  - Each block also has a tag that includes the location where the data was stored in the cache. 
  - When data is requested from the cache, 
    - a search occurs through the tags to find the specific content that's needed in level one (L1) of the memory. 
  - If the correct data isn't found, more searches are conducted in L2.
- If the data isn't found there, 
  - searches are continued in L3, then L4, and 
  - so on until it has been found, then, it's read and loaded. 
  - If the data isn't found in the cache at all, 
  - then it's written into it for quick retrieval the next time.

### Cache hit

- A cache hit describes the situation where content is successfully served from the cache. 
- The tags are searched in the memory rapidly, and when the data is found and read, it's considered a cache hit.

###  Cold, Warm, and Hot Caches

- A cache hit can also be described as cold, warm, or hot. In each of these, the speed at which the data is read is described.
  - A hot cache is an instance where data was read from the memory at the _fastest_ possible rate. 
    - This happens when the data is retrieved from L1.
  - A cold cache is the _slowest_ possible rate for data to be read, though, 
    - it's still successful so it's still considered a cache hit. 
    - The data is just found lower in the memory hierarchy such as in L3, or lower.
  - A warm cache is used to describe data that's found in L2 or L3. 
    - It's not as fast as a hot cache, 
    - but it's still faster than a cold cache. 
    - Generally, calling a cache warm is used to express that it's slower and closer to a cold cache than a hot one.

### Cache miss

- A cache miss refers to the instance when the memory is searched, and the data isn't found. 
- When this happens, the content is transferred and written into the cache.

### Cache Invalidation

- Cache invalidation is a process where the computer system declares the cache entries as invalid and removes or replaces them. 
- If the data is modified, it should be invalidated in the cache, if not, this can cause inconsistent application behavior.
- There are three kinds of caching systems:

#### Write-through cache

![write-through-cache](/contents/2023/system-design/images/system-design/chapter-I/caching/write-through-cache.png)

- Data is written into the cache and the corresponding database simultaneously.
  - **Pro**: Fast retrieval, complete data consistency between cache and storage.
  - **Con**: Higher latency for write operations.

#### Write-around cache

![write-around-cache](/contents/2023/system-design/images/system-design/chapter-I/caching/write-around-cache.png)

- Where write directly goes to the database or permanent storage, bypassing the cache.
  - **Pro**: This may reduce latency.
  - **Con**: It increases cache misses because the cache system has to read the information from the database in case of a cache miss. As a result, this can lead to higher read latency in the case of applications that write and re-read the information quickly. Read happen from slower back-end storage and experiences higher latency.

### Write-back cache

![write-back-cache](/contents/2023/system-design/images/system-design/chapter-I/caching/write-back-cache.png)

Where the write is only done to the caching layer and the write is confirmed as soon as the write to the cache completes. The cache then asynchronously syncs this write to the database.

**Pro**: This would lead to reduced latency and high throughput for write-intensive applications.

**Con**: There is a risk of data loss in case the caching layer crashes. We can improve this by having more than one replica acknowledging the write in the cache.

## Eviction policies

Following are some of the most common cache eviction policies:

- **First In First Out (FIFO)**: The cache evicts the first block accessed first without any regard to how often or how many times it was accessed before.
- **Last In First Out (LIFO)**: The cache evicts the block accessed most recently first without any regard to how often or how many times it was accessed before.
- **Least Recently Used (LRU)**: Discards the least recently used items first.
- **Most Recently Used (MRU)**: Discards, in contrast to LRU, the most recently used items first.
- **Least Frequently Used (LFU)**: Counts how often an item is needed. Those that are used least often are discarded first.
- **Random Replacement (RR)**: Randomly selects a candidate item and discards it to make space when necessary.

## Distributed Cache

![distributed-cache](/contents/2023/system-design/images/system-design/chapter-I/caching/distributed-cache.png)

A distributed cache is a system that pools together the random-access memory (RAM) of multiple networked computers into a single in-memory data store used as a data cache to provide fast access to data. While most caches are traditionally in one physical server or hardware component, a distributed cache can grow beyond the memory limits of a single computer by linking together multiple computers.

## Global Cache

![global-cache](/contents/2023/system-design/images/system-design/chapter-I/caching/global-cache.png)

As the name suggests, we will have a single shared cache that all the application nodes will use. When the requested data is not found in the global cache, it's the responsibility of the cache to find out the missing piece of data from the underlying data store.

## Use cases

Caching can have many real-world use cases such as:

- Database Caching
- Content Delivery Network (CDN)
- Domain Name System (DNS) Caching
- API Caching

**When not to use caching?**

Let's also look at some scenarios where we should not use cache:

- Caching isn't helpful when it takes just as long to access the cache as it does to access the primary data store.
- Caching doesn't work as well when requests have low repetition (higher randomness), because caching performance comes from repeated memory access patterns.
- Caching isn't helpful when the data changes frequently, as the cached version gets out of sync, and the primary data store must be accessed every time.

_It's important to note that a cache should not be used as permanent data storage. They are almost always implemented in volatile memory because it is faster, and thus should be considered transient._

## Advantages

Below are some advantages of caching:

- Improves performance
- Reduce latency
- Reduce load on the database
- Reduce network cost
- Increase Read Throughput

## Examples

Here are some commonly used technologies for caching:

- [Redis](https://redis.io)
- [Memcached](https://memcached.org)
- [Amazon Elasticache](https://aws.amazon.com/elasticache)
- [Aerospike](https://aerospike.com)


## Additional Notes



### Application server cache
- Placing a cache directly on a request layer node enables the local storage of response data.
- Each time a request is made to the service, the node will quickly return local, cached data if it exists.
- If it is not in the cache,
    - the requesting node will query the data from disk.
    - The cache on one request layer node could also be located both in memory (which is very fast)
    - and on the node’s local disk (faster than going to network storage).
- What happens when you expand this to many nodes?
    - If the request layer is expanded to multiple nodes, it’s still quite possible to have each node host its own cache.
    - However, if your load balancer randomly distributes requests across the nodes, the same request will go to different nodes, thus increasing cache misses.
    - Two choices for overcoming this hurdle are global caches and distributed caches.


### Distributed cache
- In a distributed cache, each of its nodes own part of the cached data.
- Typically, the cache is divided up using a consistent hashing function,
    - such that if a request node is looking for a certain piece of data,
    - it can quickly know where to look within the distributed cache to determine if that data is available.
- In this case, each node has a small piece of the cache, and will then send a request to another node for the data before going to the origin.
- Therefore, one of the advantages of a distributed cache is the ease by which we can increase the cache space,
    - which can be achieved just by adding nodes to the request pool.
- A disadvantage of distributed caching is resolving a missing node.
    - Some distributed caches get around this by storing multiple copies of the data on different nodes;
    - however, you can imagine how this logic can get complicated quickly,
    - especially when you add or remove nodes from the request layer.
    - Although even if a node disappears and part of the cache is lost, the requests will just pull from the origin—so it isn’t necessarily catastrophic!


### Global Cache
- A global cache is just as it sounds: all the nodes use the same single cache space.
- This involves adding a server, or file store of some sort, faster than your original store and accessible by all the request layer nodes.
- Each of the request nodes queries the cache in the same way it would a local one.
- This kind of caching scheme can get a bit complicated because it is very easy to overwhelm a single cache as the number of clients and requests increase,
    - but is very effective in some architectures
    - (particularly ones with specialized hardware that make this global cache very fast, or that have a fixed dataset that needs to be cached).
- There are two common forms of global caches depicted in the following diagram.
    - First, when a cached response is not found in the cache, the cache itself becomes responsible for retrieving the missing piece of data from the underlying store.
    - Second, it is the responsibility of request nodes to retrieve any data that is not found in the cache.
    - Most applications leveraging global caches tend to use the first type,
    - where the cache itself manages eviction and fetching data to prevent a flood of requests for the same data from the clients.
    - However, there are some cases where the second implementation makes more sense.
    - For example, if the cache is being used for very large files, a low cache hit percentage would cause the cache buffer to become overwhelmed with cache misses;
        - in this situation, it helps to have a large percentage of the total data set (or hot data set) in the cache.
    - Another example is an architecture where the files stored in the cache are static and shouldn’t be evicted.
    - This could be because of application requirements around that data latency—
        - certain pieces of data might need to be very fast for large data sets—
            - where the application logic understands the eviction strategy or hot spots better than the cache.

### Cache Invalidation
- While caching is fantastic, it does require some maintenance for keeping cache coherent with the source of truth (e.g., database).
- If the data is modified in the database, it should be invalidated in the cache, if not, this can cause inconsistent application behavior.
- Solving this problem is known as cache invalidation, there are three main schemes that are used:

#### Write-through cache: Write to both Cache and Database
- Under this scheme data is written into the cache and the corresponding database at the same time.
- The cached data allows for fast retrieval,
    - and since the same data gets written in the permanent storage,
    - we will have complete data consistency between cache and storage.
- Also, this scheme ensures that nothing will get lost in case of a crash, power failure, or other system disruptions.
- Although write through minimizes the risk of data loss,
    - since every write operation must be done twice before returning success to the client,
    - this scheme has the disadvantage of higher latency for write operations.

#### Write-around cache: Write to only Database, Update Cache on first read.
- This technique is similar to write through cache, but data is written directly to permanent storage, bypassing the cache.
- This can reduce the cache being flooded with write operations that will not subsequently be re-read,
    - but has the disadvantage that a read request for recently written data will create a “cache miss”
    - and must be read from slower back-end storage and experience higher latency.

#### Write-back cache: Write to only Cache, Update DB async later (replication)
- Under this scheme, data is written to cache alone, and completion is immediately confirmed to the client.
- The write to the permanent storage is done after specified intervals or under certain conditions.
- This results in low latency and high throughput for write-intensive applications,
    - however, this speed comes with the risk of data loss in case of a crash or other adverse event because the only copy of the written data is in the cache.

### Cache eviction policies
- Following are some of the most common cache eviction policies:
    1. First In First Out (FIFO): The cache evicts the first block accessed first without any regard to how often or how many times it was accessed before.
    2. Last In First Out (LIFO): The cache evicts the block accessed most recently first without any regard to how often or how many times it was accessed before.
    3. Least Recently Used (LRU): Discards the least recently used items first.
    4. Most Recently Used (MRU): Discards, in contrast to LRU, the most recently used items first.
    5. Least Frequently Used (LFU): Counts how often an item is needed. Those that are used least often are discarded first.
    6. Random Replacement (RR): Randomly selects a candidate item and discards it to make space when necessary.
 
### Application server cache
- Placing a cache directly on a request layer node enables the local storage of response data.
- Each time a request is made to the service, 
  - the node will quickly return local cached data if it exists. 
- If it is not in the cache, 
  - the requesting node will query the data from disk. 
- The cache on one request layer node could also be located both in memory (which is very fast) and 
  - on the node’s local disk (faster than going to network storage).
- What happens when you expand this to many nodes? 
  - If the request layer is expanded to multiple nodes, 
  - it’s still quite possible to have each node host its own cache. 
- However, if your load balancer randomly distributes requests across the nodes, the same request will go to different nodes, thus increasing cache misses. 
- Two choices for overcoming this hurdle are global caches and distributed caches.

### Content Distribution Network (CDN)
- CDNs are a kind of cache that comes into play for sites serving large amounts of static media. 
- In a typical CDN setup, a request will first ask the CDN for a piece of static media; the CDN will serve that content if it has it locally available. 
- If it isn’t available, the CDN will query the back-end servers for the file, cache it locally, and serve it to the requesting user.
- If the system we are building isn’t yet large enough to have its own CDN, 
    - we can ease a future transition by serving the static media off a separate subdomain 
    - (e.g. static.yourservice.com) using a lightweight HTTP server like Nginx, 
    - and cut-over the DNS from your servers to a CDN later.


### Cache Invalidation
- While caching is fantastic, it does require some maintenance for keeping cache coherent with the source of truth (e.g., database). 
- If the data is modified in the database, it should be invalidated in the cache; 
- if not, this can cause inconsistent application behavior.
- Solving this problem is known as cache invalidation; there are three main schemes that are used:
- **Write-through cache**: 
    - Under this scheme, data is written into the cache and the corresponding database at the same time. 
    - The cached data allows for fast retrieval 
        - and, since the same data gets written in the permanent storage, 
        - we will have complete data consistency between the cache and the storage. 
    - Also, this scheme ensures that nothing will get lost in case of a crash, power failure, or other system disruptions.
    - Although, write through minimizes the risk of data loss, 
        - since every write operation must be done twice before returning success to the client, 
        - this scheme has the **disadvantage of higher latency for write operations**.    
- **Write-around cache**: 
    - This technique is similar to write through cache, but data is written directly to permanent storage, bypassing the cache. 
    - This can reduce the cache being flooded with write operations that will not subsequently be re-read, 
        - but has the disadvantage that a read request for recently written data will create a “cache miss” 
        - and must be read from slower back-end storage and experience higher latency. 
- **Write-back cache**: 
    - Under this scheme, data is written to cache alone and completion is immediately confirmed to the client. 
    - The write to the permanent storage is done after specified intervals or under certain conditions. 
    - This results in low latency and high throughput for write-intensive applications, 
        - however, this speed comes with the risk of data loss in case of a crash or other adverse event because the only copy of the written data is in the cache.
        
### Cache eviction policies
- First In First Out (FIFO): 
    - The cache evicts the first block accessed first without any regard to how often or how many times it was accessed before.
- Last In First Out (LIFO): 
    - The cache evicts the block accessed most recently first without any regard to how often or how many times it was accessed before.
- Least Recently Used (LRU): 
    - Discards the least recently used items first.
    - One of the most common cache systems is LRU (least recently used). 
        - Design/Approach of an LRU cache
            - When the client requests resource A.
            - If A exists in the cache, we just return immediately.
            - If not and the cache has extra storage slots, 
                - we fetch resource A and return to the client. 
                - In addition, insert A into the cache.
            - If the cache is full, 
                - we kick out the resource that is least recently used and replace it with resource A.
            - The strategy here is to maximize the chance that the requesting resource exists in the cache. So how can we implement a simple LRU?
- Most Recently Used (MRU): 
    - Discards, in contrast to LRU, the most recently used items first.
- Least Frequently Used (LFU): 
    - Counts how often an item is needed. 
    - Those that are used least often are discarded first.
- Random Replacement (RR): 
    - Randomly selects a candidate item and discards it to make space when necessary.                       


### Concurrency
- Concurrency Issues falls into the classic reader-writer problem. 
    - When multiple clients are trying to update the cache at the same time, there can be conflicts. 
    - For instance, two clients may compete for the same cache slot and the one who updates the cache last wins.
- The common solution of course is using a lock. 
    - The downside is obvious – it affects the performance a lot. 
    - How can we optimize this?
- One approach is to **split the cache into multiple shards** 
    - and have a lock for each of them so that clients won’t wait for each other if they are updating cache from different shards. 
    - However, given that hot entries are more likely to be visited, certain shards will be more often locked than others.
- An alternative is to use commit logs. 
    - To update the cache, we can store all the mutations into logs rather than update immediately - **Event Store**. 
    - And then some background processes will execute all the logs **asynchronously (Event Store --> State)**. 
    - This strategy is commonly adopted in database design.


### LRU design
- An LRU cache should support the operations: 
    lookup, insert and delete. 
- Apparently, in order to achieve fast lookup, we need to use hash. 
    - By the same token, if we want to make insert/delete fast, something like linked list should come to your mind. 
    - Since we need to locate the least recently used item efficiently, we need something in order like queue, stack or sorted array.
- To combine all these analyses, 
    - we can use queue implemented by a doubly linked list to store all the resources. 
    - Also, a hash table with resource identifier as key and address of the corresponding queue node as value is needed.
- Here’s how it works. 
  - when resource A is requested, 
  - we check the hash table to see if A exists in the cache. 
  - If exists, we can immediately locate the corresponding queue node and return the resource. 
  - If not, we’ll add A into the cache. 
  - If there are enough space, we just add it to the end of the queue and update the hash table. 
  - Otherwise, we need to delete the least recently used entry. 
  - To do that, we can easily remove the head of the queue and the corresponding entry from the hash table.