---
layout: page
title: CAP
permalink: /cap/
---


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