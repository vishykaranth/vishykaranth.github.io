---
layout: page
title: Eventual Consistency 
permalink: /eventual-consistency/
---


### Eventual Consistency (EC)

- When an application makes a change to a data item on one machine, that change has to be propagated to the other replicas. 
    - Since the change propagation is not instantaneous, 
    - there’s an interval of time during which some of the copies will have the most recent change, but others won’t. 
    - In other words, the copies will be mutually inconsistent. 
    - However, the change will eventually be propagated to all the copies, and hence the term “eventual consistency”. 
    - The term eventual consistency is simply an acknowledgement that there is an unbounded delay in propagating a change made on one machine to all the other copies. 
    - Eventual consistency is not meaningful or relevant in centralized (single copy) systems since there’s no need for propagation.

- Eventual Consistency (EC) is viewed as a necessary evil most of the time. 
    - The simple CRUD unit of work, which ensures all data changes are performed successfully is deeply ingrained into our brains as developers. 
    - This is our default approach and only when dealing with distributed apps or Event Sourcing and CQRS etc we be grudgingly tolerate that eventual consistency, because we have to!
- People who view EC as normal will tell you that this is how the real world works and usually a business transaction is by its nature eventual consistent and we're doing DDD so you should accept it, like it or not! But let's see, why most real world processes are EC?
- Let's take the "buying from your favourite burger/coffe joint" example and let's pretend we're the manager and we like the CRUDy unit of work approach where every change should be part of a unit of work. The client orders, then pays with credit card,then waits for the order to be fulfilled. Let's say the payment failed. Great, the order is automatically canceled (rollback) and everyone is happy, right? Then the client tries again but paying cash this time. The order is sent again, payment is received and now we're waiting for the product. Once that's done, we consider that business transaction (process) complete. Now, imagine this process for an online shop who has to deal with shipments as well.
- What's wrong with the above approach? For starters, it's kinda stupid to cancel the order (think about the waste and lost time), just because some step in the process has temporarily failed. Secondly, each customer would have to wait (synchronously) until the process end successfully. For a popular business this means low productivity and huge lines and unhappy customers. And I can guarantee that in a matter of hours if not sooner, someone will try to optimize the process by reverting to how the process works today in the real world i.e eventually consistent.
- Clearly, when dealing with a process that involves long running operations (each domain has their own standard of what 'long' means) eventual consistency is the natural way to make the process scalable. As you can see, nothing related to things like CQRS or technical methodologies, it's just the way the domain works.
- But what if we don't deal with long running operations? And we're using Event Sourcing and CQRS i.e our technical implementation imposes the eventual consistency? Well, the reason why we choose such solutions is mainly to increase performance or scalability. And the good news is, DDD concepts like the Aggregate it still useful because it tells us what changes should be together i.e not distributed. And honestly there aren't many situations where it makes technical sense to store aggregate data in different databases.
- CQRS allows us to scale queries very easily, but there's always this issue of "What if my read model doesn't have the latest data and my command model needs it". Don't worry, in an app with high traffic, by the time you've read some data from the db, business state has already changed. Simply put you'll always be behind. For apps with moderate writes, 99% of cases you're going to deal with the latest data. And, like I've written here, what matters is the business impact , that is, if the business can wait for a couple of (milli)seconds. So, yes it depends on that specific problem of the domain.
- In conclusion, we should accept the fact that the unit of work approach is not the best solution that we should always strive for and the fact that a domain transaction is a very specific and different thing from a database transaction.

#### Basics Of Eventual consistency:
- Your data is replicated on multiple servers
- Your clients can access any of the servers to retrieve the data
- Someone writes a piece of data to one of the servers, but it wasn't yet copied to the rest
- A client accesses the server with the data, and gets the most up-to-date copy
- A different client (or even the same client) accesses a different server (one which didn't get the new copy yet), and gets the old copy
- Because it takes time to replicate the data across multiple servers, requests to read the data might go to a server with a new copy, and then go to a server with an old copy. 
    - The term "eventual" means that eventually the data will be replicated to all the servers, and thus they will all have the up-to-date copy.
- Eventual consistency is a must if you want low latency reads, 
    - since the responding server must return its own copy of the data, and 
    - doesn't have time to consult other servers and reach a mutual agreement on the content of the data. 


#### Summary 
- Strong Consistency offers up-to-date data but at the cost of high latency.
- While Eventual consistency offers low latency but may reply to read requests with stale data since all nodes of the database may not have the updated data.
- Consistency here means that a read request for an entity made to any of the nodes of the database should return the same data.
- Redundancy introduces Reliability.