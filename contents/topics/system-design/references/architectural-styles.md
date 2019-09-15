---
layout: page
title: Architectural Styles
permalink: /architectural-styles/
---
### BASE : Basically Available, Soft state, Eventual consistency

- The BASE acronym is used to describe the properties of certain databases, usually NoSQL databases. 
    - It's often referred to as the opposite of ACID.
- Basically available could refer to the perceived availability of the data. 
    - If a single node fails, part of the data won't be available, but the entire data layer stays operational.
- The BASE acronym was defined by Eric Brewer, who is also known for formulating the CAP theorem.
- The CAP theorem states that a distributed computer system cannot guarantee all of the following three properties at the same time:
    - Consistency
    - Availability
    - Partition tolerance
- A BASE system gives up on consistency.
    - Basically available indicates that the system does guarantee availability, in terms of the CAP theorem.
    - Soft state indicates that the state of the system may change over time, even without input. 
        - This is because of the eventual consistency model.
    - Eventual consistency indicates that the system will become consistent over time, given that the system doesn't receive input during that time.
    
### CAP - Brewer’s Theorem

- There are three core systemic requirements that exist in a special relationship when it comes to designing and deploying applications in a distributed environment - 
    - Consistency, 
    - Availability and 
    - Partition Tolerance.

#### Consistency

- A service that is consistent operates fully or not at all. 
    - Gilbert and Lynch use the word “atomic” instead of consistent in their proof, 
    - which makes more sense technically because, strictly speaking, consistent is the C in ACID as applied to the ideal properties of database transactions 
        - and means that data will never be persisted that breaks certain pre-set constraints. 
    - But if you consider it a preset constraint of distributed systems that multiple values for the same piece of data are not allowed then I think the leak in the abstraction is plugged 
    - (plus, if Brewer had used the word atomic, it would be called the AAP theorem and we’d all be in hospital every time we tried to pronounce it).
- In the book buying example you can add the book to your basket, or fail. 
    - Purchase it, or not. 
    - You can’t half-add or half-purchase a book. 
    - There’s one copy in stock and only one person will get it the next day. 
    - If both customers can continue through the order process to the end (i.e. make payment) the lack of consistency between what’s in stock and what’s in the system will cause an issue. 
    - Maybe not a huge issue in this case - someone’s either going to be bored on vacation or spilling soup - but scale this up to thousands of inconsistencies and give them a monetary value 
    - (e.g. trades on a financial exchange where there’s an inconsistency between what you think you’ve bought or sold and what the exchange record states) and it’s a huge issue.
- We might solve consistency by utilising a database. 
    - At the correct moment in the book order process the number of War and Peace books-in-stock is decremented by one. 
    - When the other customer reaches this point, the cupboard is bare and the order process will alert them to this without continuing to payment. 
    - The first operates fully, the second not at all.
- Databases are great at this because they focus on ACID properties and give us Consistency by also giving us Isolation, 
    - so that when Customer One is reducing books-in-stock by one, 
    - and simultaneously increasing books-in-basket by one, 
    - any intermediate states are isolated from Customer Two, 
    - who has to wait a few milliseconds while the data store is made consistent.

#### Availability

- Availability means just that - the service is available (to operate fully or not as above). 
    - When you buy the book you want to get a response, not some browser message about the web site being uncommunicative. 
    - Gilbert & Lynch in their proof of CAP Theorem make the good point that availability most often deserts you when you need it most - 
        - sites tend to go down at busy periods precisely because they are busy. 
        - A service that’s available but not being accessed is of no benefit to anyone.

#### Partition Tolerance

- If your application and database runs on one box then (ignoring scale issues and assuming all your code is perfect) your server acts as a kind of atomic processor in that it either works or doesn’t 
    - (i.e. if it has crashed it’s not available, but it won’t cause data inconsistency either).
- Once you start to spread data and logic around different nodes then there’s a risk of partitions forming. 
    - A partition happens when, say, a network cable gets chopped, and Node A can no longer communicate with Node B. 
    - With the kind of distribution capabilities the web provides, temporary partitions are a relatively common occurrence 
    - and they’re also not that rare inside global corporations with multiple data centres. 
- Partition Tolerance :: No set of failures less than total network failure is allowed to cause the system to respond incorrectly  
- one-node partition is equivalent to a server crash, because if nothing can connect to it, it may as well not be there.

#### The Significance of the Theorem

- CAP Theorem comes to life as an application scales. 
    - At low transactional volumes, small latencies to allow databases to get consistent has no noticeable affect on either overall performance or the user experience. 
    - Any load distribution you do undertake, therefore, is likely to be for systems management reasons.
- But as activity increases, these pinch-points in throughput will begin limit growth and create errors. 
    - It’s one thing having to wait for a web page to come back with a response 
    - and another experience altogether to enter your credit card details to be met with “HTTP 500 java.lang.schrodinger.purchasingerror” 
        - and wonder whether you’ve just paid for something you won’t get, 
        - not paid at all, 
        - or maybe the error is immaterial to this transaction. 
        - Who knows? 
    - You are unlikely to continue, more likely to shop elsewhere, and very likely to phone your bank.
- Either way this is not good for business. 
    - Amazon claim that just an extra one tenth of a second on their response times will cost them 1% in sales. 
    - Google said they noticed that just a half a second increase in latency caused traffic to drop by a fifth.
- Scalability 
    - whilst addressing the problems of scale might be an architectural concern, the initial discussions are not. 
        - They are business decisions. 
        - I get very tired of hearing, from techies, that such-and-such an approach is not warranted because current activity volumes don’t justify it. 
        - It’s not that they’re wrong; more often than not they’re quite correct, 
            - it’s that to limit scale from the outset is to implicitly make revenue decisions - a factor that should be made explicit during business analysis.
    - once you embark on discussions around how to best scale your application the world falls broadly into two ideological camps: 
        - the database crowd 
            - The database crowd will tend to address scale by talking of things like optimistic locking and sharding, keeping the database at the heart of things.
        - and the non-database crowd.   
            - The non-database crowd will tend to address scale by managing data outside of the database environment.          


#### Dealing with CAP

- You’ve got a few choices when addressing the issues thrown up by CAP. The obvious ones are:
    - Drop Partition Tolerance
        - If you want to run without partitions you have to stop them happening. One way to do this is to put everything (related to that transaction) on one machine, or in one atomically-failing unit like a rack. It’s not 100% guaranteed because you can still have partial failures, but you’re less likely to get partition-like side-effects. There are, of course, significant scaling limits to this.
    - Drop Availability
        - This is the flip side of the drop-partition-tolerance coin. On encountering a partition event, affected services simply wait until data is consistent and therefore remain unavailable during that time. Controlling this could get fairly complex over many nodes, with re-available nodes needing logic to handle coming back online gracefully.
    - Drop Consistency
        - Or, as Werner Vogels puts it, accept that things will become “Eventually Consistent” (updated Dec 2008). Vogels’ article is well worth a read. He goes into a lot more detail on operational specifics than I do here.
    - Lots of inconsistencies don’t actually require as much work as you’d think (meaning continuous consistency is probably not something we need anyway). In my book order example if two orders are received for the one book that’s in stock, the second just becomes a back-order. As long as the customer is told of this (and remember this is a rare case) everybody’s probably happy.
    - The BASE Jump
        - The notion of accepting eventual consistency is supported via an architectural approach known as BASE (Basically Available, Soft-state, Eventually consistent). 
        - BASE, as its name indicates, is the logical opposite of ACID, though it would be quite wrong to imply that any architecture should (or could) be based wholly on one or the other. This is an important point to remember, given our industry’s habit of “oooh shiny” strategy adoption.
- the term “BASE” was first presented in the 1997 SOSP article that you cite.  
    - BASE is contrived a bit, but so is “ACID” – much more than people realize, so we figured it was good enough. 
    - ACID was a stretch too – the A and D have high overlap and the C is ill-defined at best. But the pair connotes the idea of a spectrum, which is one of the points of the PODC lecture as you correctly point out.
            
#### Summary

- That you can only guarantee two of Consistency, Availability and Partition Tolerance is real 
    - and evidenced by the most successful websites on the planet. 
    - If it works for them I see no reason why the same trade-offs shouldn’t be considered in everyday design in corporate environments. 
    - If the business explicitly doesn’t want to scale then fine, simpler solutions are available, but it’s a conversation worth having. 
    - In any case these discussions will be about appropriate designs for specific operations
    - Different parts of the same service can choose different points in the spectrum”. 
    - Sometimes you absolutely need consistency whatever the scaling cost, because the risk of not having it is too great.
- Amazon and EBay don’t have a scalability problem. 
    - They had one and now they have the tools to address it. 
    - That’s why they can freely talk about it. 
    - Any scaling they do now (given the size they already are) is really more of the same. 
    - Once you’ve scaled, your problems shift to those of operational maintenance, monitoring, rolling out software updates etc.    
    
### The Challenges of Latency

- The time it takes packets to flow from one part of the world to another. 
- The second fallacy of distributed computing is "Latency is zero". 
- Yet so many designs attempt to work around latency instead of embracing it. 
- This is unfortunate and in fact doesn't work for large-scale systems. Why?
    - In any large-scale system, there are a few inescapable facts:
        - A broad customer base will demand reasonably consistent performance across the globe.
        - Business continuity will demand geographic diversity in your deployments.
        - It will take 30ms for a packet to traverse the Atlantic even with speed of light.
- Latency is a critical part of every system architecture. 
    - Yet making latency tolerance a first order constraint in the architecture is not that common. 
    - The result is systems that become heavily influenced by the distance between deployments. 
        - The Internet is a part of foundation of the global economy. 
        - Companies need to reliably reach their customers regardless of where they may be located. 
        - Architectures that force close geographic proximity of the components limit the quality of service provided to geographically distributed customers. 
        - Response time will obviously degrade the further customers are from the servers, but so will reliability. 
        - Despite the tremendous increase in the reliability of traffic routing on the Internet, the further you are from a service, the more often that service will be effectively unavailable to you.
- Protecting the business against disasters also requires geographically diverse locations. 
    - Natural and man-made disasters can devastate a region. 
    - Power grids cover substantial portions of the US. 
    - Witness the sheer volume of landmass that was without power during grid shutdowns in the past 10 years. 
    - If your company generates revenue from its web site, then the architecture needs to insure the site is up even in the face of localized or even broad regional events.
- One of the underlying principles is assuming high latency, not low latency. 
    - An architecture that is tolerant of high latency will operate perfectly well with low latency, but the opposite is never true.

#### Asynchronous Architecture
- The web has created an interaction style that is very problematic for building asynchronous systems. 
    - The web has trained the world to expect request/response interactions, with very low latency between the request and response. 
    - These expectations have driven architectures that are request/response oriented that lead to synchronous interactions from the browser to the data. 
    - This pattern does not lend itself to high latency connections.
- Latency tolerance can only be achieved by introducing asynchronous interactions to your architecture. 
    - The challenge becomes determining the components that can be decoupled and integrated via asynchronous interactions. 
    - An asynchronous architecture is far more than simply changing the request/response from a call to a series of messages though. 
    - The client is still expecting a response in a deterministic time. 
    - Asynchronous architectures shift from deterministic response time to probabilistic response time. 
    - Removing the determinism is uncomfortable for users and probably for your business units, but is critical to achieving true asynchronous interactions.
- It is impractical to decompose the typical web application into components that perform all external interactions asynchronously. 
    - But it should be possible to identify use cases that must support synchronous interactions and those that do not.
- Let's consider the billing component of a web site. 
    - Billing is a secondary operation as far as the community member is concerned. 
    - They have something of value the site provides and billing is a necessary evil that allows them to retain access to the value. 
    - Most billing systems will provide the user with the ability to review their balance, see account history, and make payments. 
    - Behind the scenes, fees need to be assessed, invoices generated, and automated payments processed.
    - Account balance, history, and the payment initiation would most likely be web pages. 
        - It would be difficult to transform these interactions into an asynchronous flow, as the interactions with the users would become unnatural. 
        - Notice though, I said payment initiation. 
        - Here's an example of where the interaction and expectations can be changed to become more latency tolerant.

#### Monolithic Data
- You can decompose your applications into a collection of loosely coupled components; 
    - expose your services using asynchronous interfaces, 
    - and yet still leave yourself parked in one data center with little hope of escape. 
    - You have to tackle your persistence model early in your architecture 
    - and require that data can be split along both functional and scale vectors or you will not be able to distribute your architecture across geographies. 
    - I recently read an article where the recommendation was to delay horizontal data spreading until you reach vertical scaling limits. 
    - I can think of few pieces of worse advice for an architect. 
    - Splitting data is more complex than splitting applications. 
    - But if you don't do it at the beginning, applications will ultimately take short cuts that rely on a monolithic schema. 
    - These dependencies will be extremely difficult to break in the future.    
- Partitioning data presents the modeler with several challenges. 
    - The most common is maintaining ACID compliant transactions across the data partitions. 
    - The traditional approach is to rely upon distributed transactions and two-phase commit. 
    - The problem with distributed transactions is they create synchronous couplings across the databases. 
    - Synchronous couplings are the antithesis of latency tolerant designs.
- The alternative to ACID is BASE:
    - Basically Available
    - Soft state
    - Eventually consistent
    - BASE frees the model from the need for synchronous couplings. 
        - Once you accept that state will not always be perfect and consistency occurs asynchronous to the initiating operation, 
        - you have a model that can tolerate latency. 
        - Of course there are situations where data needs to be consistent at the end of an operation. 
        - The CAP Theorem is a useful tool for determining what data to partition and what data must conform to ACID.
    - The CAP Theorem states that when designing databases you consider three properties, Consistency, Availability, and Partitioning. 
    - You can have at most two of the three for any data model. 
    - Organizing your data model around CAP allows you to make the appropriate decisions with regards to consistency and latency.
- Design for Active/Active
    - If you do a good job with the preceding recommendations, 
    - then you've most likely created an architecture that can service your customers from all of your locations simultaneously. 
    - This architecture will serve your disaster recovery needs well as it is a more efficient and responsive approach than an active/passive pattern where only one location is serving traffic at a time. 
    - Utilization of your resources will be higher and by placing services nearer your customers, you are better meeting their needs as well. 
    - Additionally, active/active designs handle localized geographic events better as traffic can simply be rebalanced from the impacted data center to your remaining data centers. 
    - Business continuity is improved.
- Traditional active/passive designs for disaster recovery have a complex testability problem that will often offset the challenges of an active/active design. 
    - Determining whether the passive data center is truly ready to take traffic can only be done by sending traffic to it. 
    - This is difficult to do in an active/passive design without impacting customers. 
    - Active/active has an inherent self-proof of suitability for managing traffic.

#### Summary
- Any aspect of your architecture that is ignored will become a problem. 
- This is a simple rule of doing design. 
- Latency is one aspect that must be considered. 
- Decompose your architecture, taking systemic qualities into consideration. 
- Decompose data, and give up on ACID levels of consistency. 
- Assume that you will have highly latent connections between your components. 
- If you take this approach, 
    - the result will be an architecture that is not only tolerant of broad geographic deployments, 
    - but is more responsive to your customers and more resilient to disasters.    
    

### A Loose Coupling Strategy

- Loose coupling is basically minimizing the dependencies between two or more components. 
    - One of many reasons why companies abandon monolithic applications, is that code in those applications mostly are very tightly coupled. 
    - Changing code on one place possibly involves making changes in other places, which increases the development time and effort needed to write that extra feature. 
    - Also separating your concerns within a monolith is very difficult to maintain, because the amount of concerns you have to deal with keeps growing.
    - With microservices, every service has it's own context and set of code and takes care of (a) specific concern. 
        - One microservice can change its entire logic from the inside, but from the outside it still does the same thing. 
        - Therefore the decoupling between microservices is enforced automatically by **separating concerns**. 
        - Microservices can communicate via, 
            - service discovery (REST API) 
            - messaging (Kafka, JMS)
            
            
#### REST API 
- REST API are endpoints in each microservice to fetch and modify data. 
- Cons
    - You need to give every microservice knowledge about the REST endpoints of other services (shared knowledge). 
    - A microservice has knowledge about the purpose of an other service and what the service possibly can do. 
    - As a result system becomes a distributed monolith, because the business logic is split up into separate contexts with still a very high reliability between each service. 
    - A high reliability between services will create issues, because a microservice that is doing calls to an unavailable REST endpoint (because of a service outage example), cannot function on it's own. 
    - This is an undesirable manner of coupling.            
    


### Scalability - Onwards and upwards

- Of all the system qualities debated by architects and developers, scalability is perhaps the one that has the greatest reputation for being mysterious and poorly understood. 
- A service is said to be scalable if, when we increase the resources in a system, it results in increased performance in a manner proportional to resources added.
- Any system can scale given enough time and money. 
    - An extreme example of this would constitute a rewrite, which though inconvenient could still provide a proportional uplift. 
    - double the CPU, double the throughput capability. 
    - And to remain truly scalable, when we double that CPU and reap the commercial rewards of handling twice as much business, 
        - we must not see a diminishing capability in other system qualities like data integrity.
- Our business runs a web site selling fruit and vegetables. 
    - In a normal day we sell 500 bananas, 
    - but recently a need has been identified for us to scale up to 1000 banana sales per day because we are launching a new campaign in the run up to Christmas, 
    - to capitalise on the seasonal trifle-fest.
    - Our board of directors can empirically say whether or not we are delivering the scale required, 
    - because in one day there will be 1000 bananas or more leaving the warehouse and not 500. 
    - Although we’ll obviously have to track uncompleted orders just to make sure that marketing and the product managers got their predictions correct and orders for 1000+ bananas were actually placed into online baskets.
- The first thing to dispense with is any form of rewriting or restructuring our platform. 
    - That can’t possibly be scalability. 
    - If we’ve written our banana-shop software in all the worst ways, 
    - we may have no choice but to respond to the business with a plan, 
    - budget and team schedule to meet their scaling needs but that would break the change-partity concept I’ve talked about before 
    - a measurably good architecture meets a small business change (selling more bananas) with a small technology change (buying more memory, for example).
    - Change, and therefore scaling, is rarely going to be free but anyone would understand the frustration of a product manager who has to watch all those banana-obsessed English people going to competitor websites because the project to enable them to buy his fruit can’t be completed in time to meet the seasonal surge.
    - Next on our dispensing list is consequential effect. 
        - There’s no sense being able to handle 1000 banana orders in one day if our customer service application becomes unstable as it receives the order data, 
        - thereby reducing our ability to handle calls about cranberry availability. 
        - That can’t possibly be scalability either.
- And finally we must understand that scalability comes in many forms. 
    - This example uses an increase in throughput (bananas per day), 
    - but maybe if we examined the requirement more closely we would find that our customers aren’t domestic trifle-makers but restaurants hoping to cater to hoards of trifle-loving office workers at the annual Christmas party. 
    - Suddenly we’re facing more fruit and vegetables per basket in one hour (because restaurants buy all their day’s supply between 6am and 7am say). 
    - Now we have to scale capacity in addition to scaling throughput.
- So a few points about scalability to bear in mind are:
    - Only local circumstances will define how your business measures ease of scaling 
    - but do make sure it is defined somewhere and not left as a debatable point for later. 
    - Clearly, most organisations will put direct and indirect cost at the top of the list, 
    - closely followed by timeliness, 
    - unless you work in the FMCG world where moving quickly can often trump the initial cost of deployment.
- Scaling, even easily, against one metric easily may not be of much value if another application or service is negatively impacted.
- How you scale can be a multi-directional thing. 
    - It’s easy to capture a requirement badly, 
    - which is why I always says you need to be careful of blindly meeting business requirements.
- Some of this may sound a little vague, 
    - but if scalability were that prescriptive then everyone would be doing it, with ease, and with little debate. 
    - I think that what makes scalability seem more dark art than simple checklist is also what makes it an engrossing design feature.
- Scalability is an enormous subject. 
    - Certainly too big to do any justice to here. 
    - A couple of good sources worth reading are Building Scalable Web Sites and sites like High Scalability. 
    - It’s also true to say that almost every design decision you make will affect the ease with which you can scale or add to the cost of doing so. 
    - But there are some basic techniques and ideas we might have adopted when we designed portal-of-fruit.com such that this banana opportunity could more easily be seized.
- Look for Queues
    - A prime enemy of scalability is the queue. 
    - Just as for cars on the roads around the holiday season, 
    - anything that causes a small queue in one place can quickly have a knock-on affect in other places. 
    - In reality, of course, there are queues and latencies all through software - if there weren’t then everything would complete in zero seconds 
    - but avoiding operations that require blocking should be a primary directive for the critical (i.e. customer affecting) portions of the business process.
    - Blocking generally takes place whenever something requires consistency in its result. 
    - For example, when you hand data over from one application to another you would expect a short pause while the receiver get’s itself into a consistent state 
    - (i.e. persists the data somewhere so that it can survive a crash). 
    - This is good for the sender because it can be confident its data is safe, 
        - but the downside is that a queue can form because no other process can use the service while the block is in place.
- Granularity
    - To reduce blocking incidents you need to do more in one place (within the same memory space, same process etc), 
    - so it’s no coincidence that discussions on the topic of transactions and consistency often focus on component scope and granularity. 
    - I’m of the school of thought that these things will, by and large, sort themselves out once some other issues are resolved.
    - One end of the consistency spectrum would be to see the banana order process as one big transaction 
    - the basket is submitted as the initiation of that transaction and, 
    - from the user’s point of view, it either succeeds or fails 
    - (failure here would be because of lack of available stock, insufficient credit, or perhaps a technical reason). 
    - And, of course, this view is true from the user’s perspective - a customer clicks a button and either gets what they want, or not.
- Transactions
    - Indeed this is closer to the perspective commonly taken by product managers. 
    - What they want is to give the customer the best possible experience, 
    - and so naturally they see the purchase process in quite literal transactional terms 
    - (they might not know they are talking about ACID, but in essence they are). 
    - To them the best possible customer experience is for the web site to go away and do all the work and then come back with a nice message for the customer saying all is well and bananas will be leaving to meet them shortly.
    - And when you analyse our banana order fulfilment business process it is tempting to draw it out as an elongated data flow diagram: 
        - validate order, 
        - if ok then check stock, 
        - if ok then reserve stock, 
        - if ok then take payment, 
        - if ok then update ERP, 
        - if ok then alert warehouse, 
        - if ok then confirm order. 
    - Clearly each activity depends on the previous one being successful for it to start and for websites with low throughput figures this approach might even work fine.
    - Every process step (reserve stock, take payment) could be and ACID transaction talking directly to a database. 
    - You would have very strong consistency and customers would indeed be happy. 
    - If you were to think in terms of services then each order step would be an atomic service, 
    - and your process control would simply call each service and get an OK (in which case it would call the next service in the workflow chain) 
    - or it would get an error (in which case it would perform some atomic rollback operations such as un-reserving the stock and return some conciliatory error message back to the customer).
- Rollback
    - And here we have identified one of the quandaries of atomic services. 
    - What if a rollback fails? We have reserved some stock successfully but, because the customer failed on payment, we have to un-reserve it successfully too. 
    - If the stock reserve/un-reserve service experiences an issue, we could end up with reserved stock that will never leave the warehouse, 
    - and we can’t get into a wait-and-try-again loop because we have a customer waiting for an answer.
    - This scenario is interesting because we have an operation that’s important to us, but not the customer. 
    - They care that we reserve stock for them, but not that we un-reserve something that they are never going to be receiving. 
    - If you think of the order process as a vector that must get from A (submitted basket) to B (happy, or at least informed, customer) then un-reserving stock is orthogonal to that process. 
    - And for scalability, orthogonal activities should be designed as asynchronous services - because they don’t block and there’s minimal latency (only long enough to acknowledge that the instruction has been received successfully).
- Orthogonality
    - So there’s an interesting idea. 
    - If we can remove operations from the order vector that are important to us but not the customer, 
    - perhaps we should ask ourselves long and hard about what really is important to the customer?
    - What about reserving stock? 
    - On the face of it, it seems logical to make this part of the critical path, but why? 
    - If we offline the entire stock control service what would be the experience? 
    - Well, we’d have to email the customer a few minutes later if we we’re experiencing a stock level issue (once the asynchronous transaction had completed) 
    - and give them a chance to cancel the order, but we’d also have an opportunity to place a back order.
    - Not only do we get better scalability, 
    - we get an additional customer service and we’ve managed to decouple order processing workflow from the stock control service so if it’s down we can still collect orders.
    - As an aside, much of the logic I am talking about here went into the development of the bespoke order management system that powered the Virgin Mobile UK website. 
    - I chose Gigaspaces as the operating platform for that system because it had another advantage that boosts scalability by a factor - grid technology that doesn’t tie consumers and suppliers together.
    - Taking the pessimistic view that any legacy system we needed to talk to might suffer from sporadic availability, 
    - the design used a shared-memory Javaspace as a repository for order documents.
    - All a process step required was an order document in the appropriate state (i.e. needing a credit check) to be collected from the space 
    - and passed to the relevant legacy application. 
    - If the application was down the document remained in the space. 
    - The user was long gone from the process, having received a confirmation email on submission of the order. 
    - If a document sat too long in the space, an alert could be raised. 
    - If a document contained data that for some reason made it incompatible with a legacy application (e.g. non-existent post code entered erroneously) this could be fixed and the document put back in the space.
    - The point being that the “consumer” simply had to put a document of the right format into the space, 
    - having no idea of the implementation details or location of the “supplier”. 
    - Moving things in and out of the space was atomic, 
    - as were the eventual writes to the database - thus, according to Brewers Theorem, 
    - we sacrificed immediate data consistency in return for improved availability of the money-making customer-satisfying processing engine and better partition-tolerance of those areas of the architecture that might not be as reliable as we need. 
    - This sacrifice is what also gives us scalability.    