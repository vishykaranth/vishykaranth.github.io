---
layout: page
title: HTTP Caching
permalink: /google-http-caching/
---

## HTTP Caching  
- The performance of web sites and applications can be significantly improved by reusing previously fetched resources. 
- Web caches reduce latency and network traffic and thus lessen the time needed to display a representation of a resource. 
- By making use of HTTP caching, Web sites become more responsive.

### Different kinds of caches
- Caching is a technique that stores a copy of a given resource and serves it back when requested. When a web cache has a requested resource in its store, it intercepts the request and returns its copy instead of re-downloading from the originating server. This achieves several goals: it eases the load of the server that doesn’t need to serve all clients itself, and it improves performance by being closer to the client, i.e., it takes less time to transmit the resource back. For a web site, it is a major component in achieving high performance. On the other side, it has to be configured properly as not all resources stay identical forever: it is important to cache a resource only until it changes, not longer.
- There are several kinds of caches: these can be grouped into two main categories: private or shared caches. A shared cache is a cache that stores responses for reuse by more than one user. A private cache is dedicated to a single user. This page will mostly talk about browser and proxy caches, but there are also gateway caches, CDN, reverse proxy caches and load balancers that are deployed on web servers for better reliability, performance and scaling of web sites and web applications.

- ![](../images/HTTPCachtType.png)