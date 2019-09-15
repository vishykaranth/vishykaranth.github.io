### Caching

#### Introduction 

- Load balancing helps you scale horizontally across an ever-increasing number of servers,
    - but caching will enable you to make vastly better use of the resources you already have as well as making otherwise unattainable product requirements feasible. 
- Caches take advantage of the locality of reference principle: recently requested data is likely to be requested again. 
- They are used in almost every layer of computing: hardware, operating systems, web browsers, web applications, and more. 
- A cache is like short-term memory: 
    - it has a limited amount of space, 
    - but is typically faster than the original data source and
    - contains the most recently accessed items. 
- Caches can exist at all levels in architecture, 
    - but are often found at the level nearest to the front end where they are implemented to return data quickly without taxing downstream levels.
    
#### Application server cache
- Placing a cache directly on a request layer node enables the local storage of response data.
- Each time a request is made to the service, the node will quickly return local cached data if it exists. 
- If it is not in the cache, the requesting node will query the data from disk. 
- The cache on one request layer node could also be located both in memory (which is very fast) and on the node’s local disk (faster than going to network storage).

    