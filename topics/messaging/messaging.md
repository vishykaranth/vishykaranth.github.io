---
layout: page
title: Messaging 
permalink: /messaging/
---


## Message queues are so useful:
- Decoupling. Producers and consumers are independent and can evolve and innovate seperately at their own rate. 
- Redundancy. Queues can persist messages until they are fully processed.
- Scalability. Scaling is acheived simply by adding more queue processors. 
- Elasticity & Spikability. Queues soak up load until more resources can be brought online. 
- Resiliency. Decoupling implies that failures are not linked. Messages can still be queue even if there's a problem on the consumer side.
- Delivery Guarantees. Queues make sure a message will be consumed eventually and even implement higher level properties like deliver at most once.
- Ordering Guarantees. Coupled with publish and subscribe mechanisms, queues can be used message ordering guarantees to consumers.
- Buffering. A queue acts a buffer between writers and readers. Writers can write faster than readers may read, which helps control the flow of processing through the entire system.  Understanding Data Flow. By looking at the rate at which messages are processed you can identify areas where performance may be improved. 
- Asynchronous Communication. Writers and readers are independent of each other, so writers can just fire and forget will readers can process work at their own leisure.