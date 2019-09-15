---
layout: page
title: Cache Basics  
permalink: /cache-basics/
---

## Caching
- Load balancing helps you scale horizontally across an ever-increasing number of servers, but caching will enable you to make vastly better use of the resources you already have as well as making otherwise unattainable product requirements feasible. 
- Caches take advantage of the locality of reference principle: recently requested data is likely to be requested again. 
- They are used in almost every layer of computing: hardware, operating systems, web browsers, web applications, and more. 
- A cache is like short-term memory: it has a limited amount of space, but is typically faster than the original data source and contains the most recently accessed items. 
- Caches can exist at all levels in architecture, but are often found at the level nearest to the front end where they are implemented to return data quickly without taxing downstream levels.
- How to design a cache system?
    - Cache system is a widely adopted technique in almost every applications today. 
    - In addition, it applies to every layer of the technology stack. 
    - For instance, at network area cache is used in DNS lookup and in web server cache is used for frequent requests.
    - A cache system stores common used resources (maybe in memory) and when next time someone requests the same resource, the system can return immediately. 
    - Cache increases the system efficiency by consuming more storage space.

## Application server cache
- Placing a cache directly on a request layer node enables the local storage of response data.
- Each time a request is made to the service, the node will quickly return local cached data if it exists. 
- If it is not in the cache, the requesting node will query the data from disk. 
- The cache on one request layer node could also be located both in memory (which is very fast) and on the node’s local disk (faster than going to network storage).
- What happens when you expand this to many nodes? If the request layer is expanded to multiple nodes, it’s still quite possible to have each node host its own cache. 
- However, if your load balancer randomly distributes requests across the nodes, the same request will go to different nodes, thus increasing cache misses. 
- Two choices for overcoming this hurdle are global caches and distributed caches.

## Content Distribution Network (CDN)
- CDNs are a kind of cache that comes into play for sites serving large amounts of static media. 
- In a typical CDN setup, a request will first ask the CDN for a piece of static media; the CDN will serve that content if it has it locally available. 
- If it isn’t available, the CDN will query the back-end servers for the file, cache it locally, and serve it to the requesting user.
- If the system we are building isn’t yet large enough to have its own CDN, 
    - we can ease a future transition by serving the static media off a separate subdomain 
    - (e.g. static.yourservice.com) using a lightweight HTTP server like Nginx, 
    - and cut-over the DNS from your servers to a CDN later.
    
## Cache Invalidation
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

### Distributed cache
- When the system gets to certain scale, we need to distribute the cache to multiple machines.
- The general strategy is to keep a hash table that maps each resource to the corresponding machine. 
    - Therefore, when requesting resource A, 
        - from this hash table we know that machine M is responsible for cache A 
        - and direct the request to M. 
        - At machine M, it works similar to local cache discussed above. 
        - Machine M may need to fetch and update the cache for A if it doesn’t exist in memory. 
        - After that, it returns the cache back to the original server.
- If you are interested in this topic, you can check more about Memcached.

LRU design
An LRU cache should support the operations: lookup, insert and delete. Apparently, in order to achieve fast lookup, we need to use hash. By the same token, if we want to make insert/delete fast, something like linked list should come to your mind. Since we need to locate the least recently used item efficiently, we need something in order like queue, stack or sorted array.

To combine all these analyses, we can use queue implemented by a doubly linked list to store all the resources. Also, a hash table with resource identifier as key and address of the corresponding queue node as value is needed.

Here’s how it works. when resource A is requested, we check the hash table to see if A exists in the cache. If exists, we can immediately locate the corresponding queue node and return the resource. If not, we’ll add A into the cache. If there are enough space, we just add a to the end of the queue and update the hash table. Otherwise, we need to delete the least recently used entry. To do that, we can easily remove the head of the queue and the corresponding entry from the hash table.

 

### Eviction policy
- When the cache is full, we need to remove existing items for new resources. 
    - In fact, deleting the least recently used item is just one of the most common approaches. 
    - So are there other ways to do that?
- The strategy is to maximizing the chance that the requesting resource exists in the cache. 
    - Random Replacement (RR) – As the term suggests, we can just randomly delete an entry.
    - Least frequently used (LFU) – We keep the count of how frequent each item is requested and delete the one least frequently used.
    - W-TinyLFU – this is modern eviction policy. 
        - The problem of LFU is that sometimes an item is only used frequently in the past, 
        - but LFU will still keep this item for a long while. 
        - W-TinyLFU solves this problem by calculating frequency within a time window. 
        - It also has various optimizations of storage.


### Summary
- Cache can be a really interesting and practical topic as it’s used in almost every system nowadays. 
