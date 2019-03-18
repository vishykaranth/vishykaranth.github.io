---
layout: page
title: Loose Coupling 
permalink: /loose-coupling/
---


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
