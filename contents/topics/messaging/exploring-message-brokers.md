## Exploring Message Brokers: RabbitMQ, Kafka, ActiveMQ, and Kestrel
### Message brokers
- Message brokers are important web-related technologies. 
- The requirements for selection of OSS message brokers: 
    - behave well when there’s a large backlog of messages, 
    - be able to create a cluster and in case of the failure of a node in a cluster, 
    - try to protect the data, but never blocks the publishers even though that might imply data lost. 
### RabbitMQ
- RabbitMQ is a well-known and popular message broker and it has many powerful features. 
- The documentation on the RabbitMQ website is excellent and there are many books available. 
- RabbitMQ is written in Erlang, not a widely used programming language but well adapted to such tasks. 
- The company Pivotal develops and maintains RabbitMQ. 
- I reviewed version 3.2.2 on CentOS 6 servers.
- The installation was easy, I installed Erlang version R14B from epel and the RabbitMQ rpm. 
- The only small issue I had was that the server is expecting “127.0.0.1″ to be resolved in /etc/hosts and the OpenStack VMs I used were missing that. 
- I also installed and enabled the management plugin.
- The RabbitMQ configuration is set in the rabbitmq.config file and it has tons of adjustable parameters. 
- In terms of client API, RabbitMQ supports a long list of languages and some standards protocols, like STOMP are available with a plugin. 
- Queues and topics can be created either by the web interface or through the client API directly. 
- If you have more than one node, they can be clustered and then, queues and topics, can be replicated to other servers.
- I created 4 queues, wrote a Ruby client and started inserting messages. 
- I got a publishing rate of about 20k/s using multiple threads but I got a few stalls caused by the vm_memory_high_watermark, 
- from my understanding during those stalls it writing to disk. 
- Not exactly awesome given my requirements. 
- Also, some part is always kept in memory even if a queue is durable so, even though I had plenty of disk space, 
- the memory usage grew and eventually hit the vm_memory_high_watermark setting. 
- The CPU load was pretty high during the load, between 40% and 50% on an 8 cores VM.
- Even though my requirements were not met, 
- I set up a replicated queue on 2 nodes and inserted a few millions objects. 
- I killed one of the two nodes and insert were even faster but then…I did a mistake. 
- I restarted the node and asked for a resync. 
- Either I didn’t set it correctly or the resync is poorly implemented but it took forever to resync and it was slowing down as it progressed. 
- At 58% done, it has been running for 17h, one thread at 100%. 
- My patience was exhausted.
- So, lots of features, decent performance, but behavior not compatible with the requirements.

### Kafka
- Kafka has been designed originally by LinkedIn, 
    - it is written in Java and it is now under the Apache project umbrella. 
    - Sometimes you look at a technology and you just say: wow, this is really done the way it should be. 
    - At least I could say that for the purpose I had. 
    - What is so special about Kafka is the architecture, 
        - it stores the messages in flat files and consumers ask messages based on an offset. 
        - Think of it like a MySQL server (producer) saving messages (updates SQL) to its binlogs and slaves (consumers) ask messages based on an offset. 
        - The server is pretty simple and just don’t care about the consumers much. 
        - That simplicity makes it super fast and low on resource. 
        - Old messages can be retained on a time base (like expire_logs_days) and/or on a storage usage base.
- So, if the server doesn’t keep track of what has been consumed on each topic, 
    - how can you have multiple consumers? 
    - The missing element here is Zookeeper. 
    - The Kafka server uses Zookeeper for cluster membership and routing while the consumers can also use Zookeeper or something else for synchronization. 
    - The sample consumer provided with the server uses Zookeeper so you can launch many instances and they’ll synchronize automatically. 
    - For the ones that doesn’t know Zookeeper, it is a highly-available synchronous distributed storage system. 
    - If you know Corosync, it provides somewhat the same functionality.
- Feature-wise, Kafka isn’t that great. 
    - There’s no web frontend built-in, 
    - although a few are available in the ecosystem. 
    - Routing and rules are in-existent and stats are just with JMX. 
- But, the performance…I reached a publishing speed of **165k messages/s** over a single thread, 
    - and I didn’t bother tuning for more. 
    - Consuming was essentially disk bound on the server, 3M messages/s… amazing. 
    - That was without Zookeeper coordination. 
    - Memory and CPU usage were modest.
- To test clustering, 
    - I created a replicated queue, inserted a few messages, stopped a replica, inserted a few millions more messages and restarted the replica. I took only a few seconds to resync.

So, Kafka is very good fit for the requirements, stellar performance, low resource usage and nice fit with the requirements.

ActiveMQ
ActiveMQ is another big player in the field with an impressive feature set. ActiveMQ is more in the RabbitMQ league than Kafka and like Kafka, it is written in Java. HA can be provided by the storage backend, levelDB supports replication but I got some issues with it. My requirements are not for full HA, just to make sure the publishers are never blocked so I dropped the storage backend replication in favor of a mesh of brokers.

My understanding of the mesh of brokers is that you connect to one of the members and you publish or consume a message. You don’t know on which node(s) the queue is located, the broker you connect to knows and routes your request. To further help, you can specify all the brokers on the connection string and the client library will just reconnect to another if the one you are connected to goes down. That looks pretty good for the requirements.

With the mesh of brokers setup, I got an insert rate of about 5000 msg/s over 15 threads and a single consumer was able to read 2000 msg/s. I let it run for a while and got 150M messages. At this point though, I lost the web interface and the publishing rate was much slower.

So, a big beast, lot of features, decent performance, on the edge with the requirements.

Kestrel
Kestrel is another interesting broker, this time, more like Kafka. Written in Scala, the Kestrel broker speaks the memcached protocol. Basically, the key becomes the queue name and the object is the message. Kestrel is very simple: queues are defined in a configuration file but you can specify, per queue, storage limits, expiration, and behavior when limits are reached. With a setting like “ discardOldWhenFull = true ”, my requirement of never blocking the publishers is easily met.

In term of clustering, Kestrel is a bit limited but each can publish its availability to Zookeeper so that publishers and consumers can be informed of a missing server and adjust. Of course, if you have many Kestrel servers with the same queue defined, the consumers will need to query all of the brokers to get the message back and strict ordering can be a bit hard.

In terms of performance, a few simple bash scripts using nc to publish messages easily reached 10k messages, which is very good. The rate is static over time and likely limited by the reconnection for each message. The presence of consumers slightly reduces the publishing rate but nothing drastic. The only issue I had was when a large number of messages expired, the server froze for some time but that was because I forgot to set maxExpireSweep to something like 100 and all the messages were removed in one pass.

So, fairly good impression on Kestrel, simple but works well.

Conclusion
For the requirements given by the customer, Kafka was like a natural fit. It offers a high guarantee that the service will be available and non-blocking under any circumstances. In addition, messages can easily be replicated for higher data availability. Kafka performance is just great and resource usage modest.