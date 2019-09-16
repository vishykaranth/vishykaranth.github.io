---
layout: page
title: Twitter System Design 2
permalink: /twitter-system-design-2/
---


### Twitter Numbers
- 200ms latency for timeline generation
- 28B tweet views per day
    - most/half? has video/photo
    - can't just query db every time
- 28B daily tvs / 200M users = 140 daily page views on average
- 1 SQL DB query * 10ms = 10ms
- 1 NoSQL queries * 1ms = 1ms per query
- 1 http request/response * 100ms = 100ms per request
- 1 query + 1 http request = 110ms
    - + http response = 201-210ms total per requset
- latency cost: 80-150ms
- Complete page rendering on browser = 5 seconds = 5000 ms
    - + request time = 5.35 seconds total
- 28B queries / 86400 seconds per day
    - = 324,074 queries per second per day (324 SQL servers @ 1kQPS)

System Design:
 - cache all static assets (js, css, html files) via CDN
     - replicate geographically
     - Akamai (reddit-style) etc.
 - app-side caching
    - to avoid cache misses, avoid each node having a cache (global cache, Memcached/Redis)
     - tweets
         - latests, hot tweets, retweets
     - session information
 - DB storage
 - Scaling out
     - replication to support a read-heavy app
- Partitioning
    - Vertical? (users, tweets, etc)
    - Sharding
        - shard by popularity of tweeters. Dedicated nodes for users with high number of followers
- Load balancer cluster
    - load balancer in front of application servers to distribute load
    - consistent hashing to randomize
    - load balancer between application server and storage?

