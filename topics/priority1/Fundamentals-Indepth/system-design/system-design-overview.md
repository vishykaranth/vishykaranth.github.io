---
layout: page
title: System - Design Overview 
permalink: /system-design-overview/
---

## System design questions in interviews
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