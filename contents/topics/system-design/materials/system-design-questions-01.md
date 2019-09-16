# System design

### Faas concepts

1. Event driven system - serverless function usually glue the services together and work properly.
2. Serverless function was called service bus in the past.
3. The features of serverless function:
   1. short-lived
   2. not a daemon
   3. stateful
   4. execute in few second
   5. make use of existing services
   
Further reading:

* [introducing-functions-as-a-service](https://blog.alexellis.io/introducing-functions-as-a-service/)
* [AWS re:Invent 2017: GitHub to AWS Lambda](https://youtu.be/lYYLGBdFXqM)
* [Local Testing and Deployment Best Practices for Serverless Applications](https://youtu.be/QRSc1dL-I4U)
* [Building a Pipeline for Your Serverless Application](https://docs.aws.amazon.com/lambda/latest/dg/build-pipeline.html)
* [AWS Lambda (Part 1): Introduction](https://youtu.be/44Bp9g7W7N8)
* [AWS CodeStar: Develop, Build, and Deploy applications on AWS](https://youtu.be/7UygHxU9fU8)
* [Serverless Architecture Patterns and Best Practices](https://youtu.be/_mB1JVlhScs)
   
### Load balancer

1. Load balancers distribute incoming client requests to computing resources:
   1. application (web app)
   2. database
2. Load balancers tasks include:
   1. Prevent requests to **unhealthy** services
   2. Prevent overloading of the service
   3. Eliminating signle point failure
   4. SSL Termination (Remove the need to install certification to each server)
   5. Session persistent
3. Route traffic algorithm
   1. Round-Robin
   2. Least Load
   3. Random
4. Layer 4 (transport layer) load balancer: NAT
5. Layer 7 (application layer) load balancer: Nginx, HAProxy, ...
6. Layer 7 load balancer redirect traffic to the service, for example:
   1. video stream to video streamming server
   2. billing request to another security server
7. Load balancer could be bottleneck (disadvantage)

### Scaling

1. Horizontal Scaling (scale out)
   1. Advantage: Load balancer helps horizontal scaling
   2. Disadvantage:
      1. Increase system complexity
      2. Server should be stateless
      3. Downstream servers such as caches and databases need to handle more simultaneous connections as upstream servers scale out
2. Vertical scaling (scale up): Upgrade the hardware to a single machine

### CAP theorem

* **Consistency** - A read is guaranteed to return the most recent write for a given client

![cap-cp](https://s3-ap-northeast-1.amazonaws.com/gist-note-assets/system-design/cap_cp.png)

* **Availability** - A **non-failing** node will return a reasonable response within a resonable amount of time (no error or timeout)

![cap-ap](https://s3-ap-northeast-1.amazonaws.com/gist-note-assets/system-design/cap_ap.png)

* **Partition Tolerance** - The system will **continue** to function **when network partitions occur**
 

Further reading:

* [CAP Theorem: Revisited](http://robertgreiner.com/2014/08/cap-theorem-revisited/)
* [A plain english introduction to CAP Theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem/)

### Consistency patterns

* Weak consistency: After a write, reads may or may not see it.
* Eventual consistency: After a write, reads will eventually see it. Data is replicated asynchronously
* Strong consistency: After a write, reads will see it. Data is replicated synchronously.

### Redis

1. Redis in memory store, 
2. A key-value store
3. Redis is a data structure serve, no query language.
4. 2 options for persistent
   1. regular snapshotting
   2. append-only files.
5. Use cases:
   1. Show latest items listings in your home page
   2. Order by user votes and time
   3. Implement expires on items
   4. Counting stuff
   5. Pub/Sub
   6. Queues
   7. Caching
   
   
Use case:

* Chat or Twitter

![chat-twitter](https://s3-ap-northeast-1.amazonaws.com/gist-note-assets/system-design/chat_twitter.png)

* Cache pattern

![cache-pattern](https://s3-ap-northeast-1.amazonaws.com/gist-note-assets/system-design/cache_pattern.png)
   
Further reading:

* [11 Common Web Use Cases Solved In Redis](http://highscalability.com/blog/2011/7/6/11-common-web-use-cases-solved-in-redis.html)
   
### Relational database management system (RDBMS)

1. ACID is a set of properties of relational database transactions.
   1. **Atomicity** - Each transaction is all or nothing
   2. **Consistency** - Any transaction will bring the database from one valid state to another
   3. **Isolation** - Executing transactions concurrently has the same results as if the transactions were executed serially
   4. **Durability** - Once a transaction has been committed, it will remain so
2. Scale a relational database

* master-slave replication
   
![master-slave-replication](https://s3-ap-northeast-1.amazonaws.com/gist-note-assets/system-design/master-slave-replica.png)
   
* master-master replication
   
![master-master-replication](https://s3-ap-northeast-1.amazonaws.com/gist-note-assets/system-design/master-master-replica.png)   
   
* federation: Federation (or functional partitioning) splits up databases by function. For example, instead of a single, monolithic database, you could have several databases
* sharding: Sharding distributes data across different databases such that each database can only manage a subset of the data
* denormalization: Denormalization attempts to improve read performance at the expense of some write performance. Redundant copies of the data are written in multiple tables to avoid expensive joins


### NoSQL

#### Type

1. key-value store
2. document-store
3. wide column store
4. graph database

Most NoSQL stores lack true ACID transactions and favor eventual consistency. In comparison with the CAP Theorem, BASE chooses **availability over consistency**

BASE is often used to describe the properties of NoSQL databases.

1. Basically available - the system guarantees availability
2. Soft state - the state of the system may change over time, even without input
3. Eventual consistency - the system will become consistent over a period of time, given that the system doesn't receive input during that period

Futher reading:

* [Scalability, Availability & Stability Patterns](https://www.slideshare.net/jboner/scalability-availability-stability-patterns)
   
### Message Queue

1. Asynchronous workflows help reduce request times for expensive operations
2. Workflow:
   1. An application publishes a job to the queue, then notifies the user of job status
   2. A worker picks up the job from the queue, processes it, then signals the job is complete
3. The user is not blocked and the job is processed in the background
4. Message queue project:
   1. Redis
   2. RabbitMQ
   3. Amazon SQS
5. Disadvantage:
   1. Introducing queues can add delays and complexity
6. Back pressure
   1. If queue growth too quick which exceed the system loading, introducing a back-pressure (the limitation of queue) can eliminate system loading. Once the queue fills up, the system can reponse a busy error code such as HTTP 503 to ask client try again later.
   
### REST vs. RPC

Futher reading:

* [Do you really know why you prefer REST over RPC?](https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/)

### Common System Architecture

#### 3 Tier Layer

* Strengths:
  * Rich frontend framework
  * Scalable (the benefit comes from stateless of app)
  * Scalable data (usually use NoSQL database)
* State in the middle tier (Since apps usually stateless)

![3-tier](https://s3-ap-northeast-1.amazonaws.com/gist-note-assets/system-design/3-tier-arch.png)

#### Shard

* Strengths:
  * Client isolation is easy
  * Simple technology
* Weekness
  * Complexity
  * Hard to understand data
  * Oversize shard (redundent shard)
* Example:
  * Slack (apps, channels)

![shard](https://s3-ap-northeast-1.amazonaws.com/gist-note-assets/system-design/sharding-arch.png)


### Video

* [Four Distributed Systems Architectural Patterns by Tim Berglund ](https://youtu.be/tpspO9K28PM)
* [Distributed Systems in One Lesson by Tim Berglund](https://youtu.be/Y6Ev8GIlbxc)


## Reference

* [system-design-primer](https://github.com/donnemartin/system-design-primer#key-value-store)
* [Back-End-Developer-Interview-Questions](https://github.com/arialdomartini/Back-End-Developer-Interview-Questions)