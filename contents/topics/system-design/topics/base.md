---
layout: page
title: BASE
permalink: /base/
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