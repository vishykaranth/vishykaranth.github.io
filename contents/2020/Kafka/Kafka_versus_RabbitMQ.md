---
layout: page
title: Kafka versus RabbitMQ
permalink: /Kafka-versus-RabbitMQ/
---

## Kafka versus RabbitMQ
- Publish/subscribe is a distributed interaction paradigm well adapted to the deployment of scalable and loosely coupled systems.
- Apache Kafka and RabbitMQ are two popular open-source and commercially-supported pub/sub systems that have been around for almost a decade and have seen wide adoption. 
- Internet has considerably changed the scale of distributed systems. 
- Distributed systems now involve thousands of entities 
    - "potentially distributed all over the world" whose location and behavior may greatly vary throughout the lifetime of the system. 
- These constraints underline the need for more flexible communication models and systems that reflect the dynamic and decoupled nature of the applications. 
- Individual point-to-point and synchronous communications lead to rigid and static applications, 
    - and make the development of dynamic large-scale applications cumbersome. 
- To reduce the burden of application designers, 
    - the glue between the different entities in such large-scale settings should rather be provided by a dedicated middleware infrastructure, 
    - based on an adequate communication scheme. 
- The publish/subscribe interaction scheme provides the loosely coupled form of interaction required in such large scale settings.
- Apache Kafka and RabbitMQ are two popular open-source and commercially-supported pub/sub systems that have been around for almost a decade 
    - and have seen wide adoption in enterprise companies.
- Despite commonalities, these two systems have different histories and design goals, and distinct features. 
    - For example, they follow different architectural models: 
        - In RabbitMQ, producers publish (batches of) message(s) with a routing key to a network of exchanges where routing decisions happen, 
        - ending up in a queue where consumers can get at messages through a push (preferred) or pull mechanism. 
- In Kafka producers publish (batches of) message(s) to a disk based append log that is topic specific. 
    - Any number of consumers can pull stored messages through an index mechanism.
- Given the popularity of these two systems and the fact that both are branded as pub/sub systems, 
    - two frequently asked questions in the relevant online forums are: 
        - how do they compare against each other and 
        - which one to use?
        
- While one can find several ad-hoc recommendations (some based on pointed benchmarks) on
the web, we found these hard to generalize to other applications and believe they do not do justice
to the systems under discussion. More specically, (i) the bigger context of such analysis, e.g.,
qualitative comparison or the distinct features of each system is oen overlooked, (ii) due to the
fast pace of developments, in particular in the case of Kaa, some of the reported results are stale
and no longer valid, and (iii) it is dicult to compare results across dierent experiments.
In this paper, we frame the arguments in a more holistic approach. More concretely, we start
(in Section 2) with a brief description of the core functionalities of publish/subscribe systems as
well as the common quality-of-service guarantees they provide. We then (in Section 3) give a
high-level description of both Apache Kaa and RabbitMQ. Based on the framework established in
Section 2, we venture into a qualitative (in Section 4) and quantitative (in Section 5) comparison of
the common features of the two systems. In addition to the common features, we list the important
features that are unique to either of the two in Section 6. We next enumerate a number of use case
classes that are best-suited for Kaa or RabbitMQ, as well as propose options for a combined use
of the two systems in Section 7. ere, we also propose a determination table to help choose the
best architecture when given a particular set of requirements.        
