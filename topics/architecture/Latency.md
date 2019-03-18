---
layout: page
title: Latency
permalink: /latency/
---


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