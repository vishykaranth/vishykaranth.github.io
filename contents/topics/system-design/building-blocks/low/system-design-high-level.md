---
layout: page
title: System - Design Overview 
permalink: /system-design-overview/
---

### Getting Started 
- A : Ask good questions
- B : Don't use buzzwords
- C : Clear and organized thinking
- D : Drive discussions with 80-20 rule

### Things to consider
- Features
- API
- Availability
- Latency
- Scalability
- Durability
- Class Diagram
- Security and Privacy
- Cost-effective

### System design questions in interviews
- Be absolutely sure you understand the problem being asked, clarify on the onset rather than assuming anything
- Use-cases. This is critical, you MUST know what is the system going to be used for, what is the scale it is going to be used for. Also, constraints like requests per second, requests types, data written per second, data read per second.
- Solve the problem for a very small set, say, 100 users. This will broadly help you figure out the data structures, components, abstract design of the overall model.
- Write down the various components figured out so far and how will they interact with each other.
- As a rule of thumb remember at least these :
    - processing and servers
    - storage
    - caching
    - concurrency and communication
    - security
    - load balancing and proxy
    - CDN
- Cost 
    - eg. What kind of DB (Is Postgres enough, if not why?), 
    - do you need caching and how much, 
    - is security a prime concern?
- Special questions 
    - Say designing a system for storing thumbnails, will a file system be enough? 
    - What if you have to scale for facebook or google? 
    - Will a nosql based database work?
- After I have my components in place, what I generally try to do is look for minor optimization in various places according to the use-cases, 
    - various tradeoffs that will help in better scaling in 99% cases.
- Scaling 
- Check with the interviewer is there any other special case he is looking to solve? 
    - Also, it really helps if you know about the company you are interviewing with, 
        - what its architecture is, 
        - what will the interviewer have more interest in based on the company and what he works on?       
           
## Common Design questions
- It generally depends what you are and you will be working on. 
- Also what your level is but these are some of the more frequent interview questions.
- Design amazon's frequently viewed product page (eg. which shows the last 5 items you saw)
- Design an online poker game for multiplayer. Solve for persistence, concurrency, scale. Draw the ER diagram for this
- Design a [url compression system] (http://www.hiredintech.com/system-design/the-system-design-process/)
Search engine (generally asked with people who have some domain knowledge): basic crawling, collection, hashing etc. Depends on your expertise on this topic
- Design dropbox's architecture. good talk on this
- Design a picture sharing website. How will you store thumbnails, photos? Usage of CDNS? caching at various layers etc.
- Design a news feed (eg. Facebook , Twitter): news feed
- Design a product based on maps, eg hotel / ATM finder given a location.
- Design malloc, free and garbage collection system. What data structures to use? decorator pattern over malloc etc.
- Design a site like junglee.com i.e price comparision, availability on e-commerce websites. When and will you cache, how much to query, how to crawl efficiently over e-commerce sites, sharding of databases, basic database design
A web application for instant messaging, eg whatsapp, facebook chat. Issues of each, scaling problems, status and availability notification etc.
- Design a system for collaborating over a document simultaneously (eg google docs)
(very common:) top 'n' or most frequent items of a running stream of data
- Design election commission architecture : Let's say we work with the Election Commission. On Counting day, we want to collate the votes received at the lakhs of voting booths all over the country. Each booth has a voting machine, which, when connected to the network, returns an array of the form {[party_id, num_votes],[party_id_2, num_votes_2],...}. We want to collect these and get the current scores in real time. The report we need continuously is how many seats is each party leading in. Please design a system for this.
- Design a logging system (For web applications, it is common to have a large number of servers running the same application, with a load balancer in front to distribute the incoming requests. In this scenario, we want to check and alarm in case an exception is thrown in any of the servers. We want a system that checks for the appearance of specific words, "Exception", "Disk Full" etc. in the logs of any of the servers. How would you design this system?)        



## System Design Problems
- Designing a URL Shortening service like TinyURL
- Designing Pastebin
- Designing Instagram
- Designing Dropbox
- Designing Facebook Messenger
- Designing Twitter
- Designing Youtube or Netflix
- Designing Typeahead Suggestion
- Designing an API Rate Limiter
- Designing Twitter Search
- Designing a Web Crawler
- Designing Facebook’s Newsfeed
- Designing Yelp or Nearby Friends
- Designing Uber backend
- Design Ticketmaster (*New*)


## Glossary of System Design Basics
- System Design Basics
- Key Characteristics of Distributed Systems
- Load Balancing
- Caching
- Data Partitioning
- Indexes
- Proxies
- Redundancy and Replication
- SQL vs. NoSQL
- CAP Theorem
- Consistent Hashing
- Long-Polling vs WebSockets vs Server-Sent Events



### Concepts to know
- Vertical vs horizontal scaling
- CAP theorem
- ACID vs BASE
- Partitioning/Sharding 
- Consistent Hashing
- Optimistic vs pessimistic locking
- Strong vs eventual consistency
- RelationalDB vs NoSQL
- Types of NoSQL
    - Key value
    - Wide column
    - Document-based
    - Graph-based
- Caching
- Data center/racks/hosts
- CPU/memory/Hard drives/Network bandwidth
- Random vs sequential read/writes to disk
- HTTP vs http2 vs WebSocket
- TCP/IP model
- ipv4 vs ipv6
- TCP vs UDP
- DNS lookup
- Http & TLS
- Public key infrastructure and certificate authority(CA)
- Symmetric vs asymmetric encryption
- Load Balancer
- CDNs & Edges
- Bloom filters and Count-Min sketch
- Paxos 
- Leader election
- Design patterns and Object-oriented design
- Virtual machines and containers
- Pub-sub architecture 
- MapReduce
- Multithreading, locks, synchronization, CAS(compare and set)

### Tools
- Cassandra
- MongoDB/Couchbase
- Mysql
- Memcached
- Redis
- Zookeeper
- Kafka
- NGINX
- HAProxy
- Solr, Elastic search
- Amazon S3
- Docker, Kubernetes, Mesos
- Hadoop/Spark and HDFS

### Approach
- Ask as many questions as possible to get a clear understanding of the requirements.
- Write them down the main use cases on the whiteboard.
- Ask re-clarifying and scope related questions.
- Once you have the requirements written down draw up a flow diagram with various components and how they are connected and talk about the possible use cases.
- Think out loud. Verify your assumptions with the interviewer.
- Be prepared to write up the APIs specs for the flows and split the applications down by features.
- Also, you would need to have a fair understanding of various types of databases. When do you use RDBMS vs NoSQL? When do you use a message queue? Which type of queue would you use?
- Detailing what kind of cache you would use and where.
- Talk about scaling, Redundancy, DR.

### Examples 
- Design a URL shortening service like bit.ly.
- How would you implement the Google search?
- Design a client-server application which allows people to play chess with one another.
- How would you store the relations in a social network like Facebook and implement a feature where one user receives notifications when their friends like the same things as they do?
- Designing an elevator system
- Design a Parking lot system
- Shopping cart – How do you store this information when you use multiple servers that are load balanced.
- How would you design a Twitter Feed? –Grokking the System Design Interview
- Recommendation system for fashion/clothes and accessories – Fundamentals here.
- How does Uber Store & retrieve lat &long for a cab driver?
- If a user is at x,y give me five of the closest drivers. – Grokking the System Design Interview
- Extend the product page X and add the auction capability.
- How are your ensuring security or localization on a mobile device?
- Design a web-based email system.
- Describes pieces, components, design, large scale, and use case
- Design an application like Siri, Cortana or Alexa
- Design Facebook or the privacy features in Facebook.
- Explain different performance scenarios for Instagram architecture.
- Explain the different places you have caching in OneDrive.
- Designing an activity feed system – Grokking the System Design Interview
- Design WhatsApp / Facebook Messenger: Issues of each, scaling problems, offline/online users and availability, notification etc – Grokking the System Design Interview” & Link-1 & Link 2
- An airline carrier is losing a lot of bags – Design a solution.
- Design Dropbox etc. – Grokking the System Design Interview
- Design X’s frequently viewed product page shows the last 5 items you viewed.
- Design the product recommendation feature based on a user’s purchase history.
- Design an online poker game or Tick Tack Toe for multi-players.
- Solve for persistence, concurrency, scale.
- Instagram –Grokking the System Design Interview
- Design a URL compression system – Bitly – Link 1, Link 2
- Search engine: basic crawling, collection, hashing etc. (Depends on your expertise on this topic).
- Autocomplete / Typeahead Search- Link
- Design a coupon system for a website like Peach or Uber.
- Design a picture sharing website. How will you store thumbnails, photos? Usage of CDNS? Caching at various layers etc.
- Design a push and inbox messaging platform.
- Design a product based on maps, eg Hotel / ATM finder given a location.
- Design malloc, free and garbage collection system. What data structures to use? Decorator pattern over malloc etc.
- Design a site like www.Pronto.com (price comparison, availability on eCommerce websites)
- When and will you cache, how often would you query, crawl efficiency, etc?
- Design a system for collaborating over a document simultaneously (e.g.: google docs)?
- Design an electronic election / Ballot machine architecture
- Design a logging system – Splunk or ELK
- Design Netflix, Youtube, Spotify –Grokking the System Design Interview
- Build a machine learning system to detect if a fake user.
- How do you design a system with 99.999% availability
- Design an amusement Park Ticketing system for user ride efficiency
- Design Uber –Grokking the System Design Interview
- Design an Inventory Management System
- Design a Video Conferencing application. – InfoQ Solution
- Design any of the above architectures only using AWS, GPC or Azure- For Any cloud team.
- Troubleshoot a slow website or a slow e-reading device.
- Instagram, TripAdvisor, Salesforce.com, StackOverflow, and many others.
- Distributed Database System Key Value Store

### Tools 
- Caching
- Memcache
- Memcached
- A comparative study of distributed caches
- Redis
- Memcached or Riak	Link	Medium
- Persistent & Ephemeral Data	Link	Low
- CDN	Link	High
- API
- Basic HTTP Response Codes	Link	High
- REST Standards	Link	High
- REST II	Link	High
- RESTful.Web.Services Book	Link	High
- Rest vs SOAP	Link	High
- API Idempotence – I	Link	Medium
- API Idempotence – II	Link	Medium
- Immutable Services	Link	Low
- Semaphore and Mutex Simplified	Link	High
- Microservices	Link	High
- Understanding REST Headers and Parameters	Link	Medium
- One API, Many Facades?	Link	Medium
- Pattern: Backends For Frontends	Link	Low
- BFF @ SoundCloud	Link	Low
- Scalability
- Throughput vs latency	Link	High
- Capacity Planning	Link	High
- Fault Tolerance, Redundant systems vs High Availability Systems	Link	High
- Apache Mesos & Docker	Link	Medium
- Ring pop I	Link	Low
- Ring pop II	Link	Low
- Architecture In General
- OAUTH	Blog	High
- Certificates and HTTPS	Link 1, Link 2	High
- TCP/IP and Networking Fundamentals	Link	High
- Scalability Harvard Web Development	Link	High
- Starvation and Deadlock I	Link	High
- Starvation and Deadlock II	Link	High
- Why stateless applications are always the way to go	Link	High
- Processes, Synchronization & Deadlock	Link	Medium
- Computer Networking	Link	Medium
- Stateless by Stan Hanks	Link	Medium
- Big Data
- MapReduce	Link	High
- Hadoop Basics	Link	High
- Machine Learning	Link	Medium
- Apache Spark (Real Time Processing of data)	Link	Low
- Parquet	Link	Low
- Queuing
- Kafka I	Link	High
- Kafka II	Link	High
- Kafka II	Link	High
- Building a Real-time Data Pipeline: Apache Kafka at LinkedIn	Link	High
- ActiveMQ	Link	Medium
- Kafka vs Rabbitmq vs Activemq vs Redis	Link	Medium
- MQueue	Link	Low
- Two strategies for Feed systems	Link	Medium
- Spotify’s Event Delivery – The Road to the Cloud	Link	Low
- HTTP Long poll Socket	Link	Low
- Pub-Sub	Link	Low
- Pub-Sub with Websphere	Link	Low
- Databases	
- Cassandra	
- NoSQL Distilled – Book
- High
- mongo DB	High
- Graph DB	High
- CAP Theorem	High
- Introduction to NoSQL by Martin Fowler	Link	High
- Other
- JASON Formats and Documentation	Link	Low
- Operating systems	Link	Low
- RPC protocol	Link	Low
- Types of pagination – Offset and cursor	Link	Medium
- Design Patterns	Link	Low
- Design Patterns Pluralsight Course	Link	Low
- Algorithms and Data Structures – Part 1	Link	Low
- Big O	Link	Medium
- Video
- Video Scalability	Link	Low
- SPI H.323 (Video Protocols)	Link	Low
- SPI H.323 (Video Protocols)	Link	Low
- SPI H.323 (Video Protocols)	Link	Low
- Recommendations Systems
- Global Recommendations	Link	Medium
- Recommendations Systems	Link	Medium
- Netflix launching in various countries & What it takes	Link	Medium
- Hash Table
- Hash Table – I	Link	High
- Hash Table – II	Link	High
- Hash Table – III	Link	Medium
- Hash Table – IV	Link	Medium
- Hash Table – V	Link	Medium
- Perfect Universal Hashing	Link	Low
- Cross Datacenter
- Inter Datacenter usage? Cassandra	Link	Medium
- Transactions Across Datacenters	Link	Medium
- Distributed Transaction Layer: App Engine	Link	Medium
- Other Interesting Things
- Twitter by Hired in tech	Link	High
- DropBox scaling	Link	High
- Interviewing at Google	Link	Low

### References 
- https://github.com/donnemartin/system-design-primer
