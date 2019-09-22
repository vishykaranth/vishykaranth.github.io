## Experience  
- **SecTM** Business Isolation maps to Sharding 
- **SecTM** distributed architecture maps to Mediator Pattern 
- **SecDB Ring** - Shard-ing region based
- JOLT - equivalent of distributed transaction across DB and Messaging - **Payment Project @ GS**
- Event Driven Architecture for **Prime Broker** and **Overnight Funding** enhancements 
- Event Storming sessions for MTF P&L and Revenue Allocation 
- Event Sourcing, Snapshot, State Recovery using snapshot, Event Source versioning for **Asset Ledger & Balance**   

## Misc - Thoughts 
- Little-Endian and Big-Endian in Java  

##Canary release
- Canary release is a technique to reduce the risk of introducing a new software version in production by slowly rolling out the change to a small subset of users before rolling it out to the entire infrastructure and making it available to everybody.
- Similar to a BlueGreenDeployment, you start by deploying the new version of your software to a subset of your infrastructure, to which no users are routed.
    - o
    - ![](../imgs/canary-release-1.png)
    - o 
- When you are happy with the new version, you can start routing a few selected users to it. There are different strategies to choose which users will see the new version: a simple strategy is to use a random sample; some companies choose to release the new version to their internal users and employees before releasing to the world; another more sophisticated approach is to choose users based on their profile and other demographics.
    - o
    - ![](../imgs/canary-release-2.png)
    - o     
- As you gain more confidence in the new version, you can start releasing it to more servers in your infrastructure and routing more users to it. A good practice to rollout the new version is to repurpose your existing infrastructure using PhoenixServers or to provision new infrastructure and decommission the old one using ImmutableServers.
- Canary release is an application of ParallelChange, where the migrate phase lasts until all the users have been routed to the new version. At that point, you can decomission the old infrastructure. If you find any problems with the new version, the rollback strategy is simply to reroute users back to the old version until you have fixed the problem.
    - o
    - ![](../imgs/canary-release-3.png)
    - o       
- A benefit of using canary releases is the ability to do capacity testing of the new version in a production environment with a safe rollback strategy if issues are found. By slowly ramping up the load, you can monitor and capture metrics about how the new version impacts the production environment. This is an alternative approach to creating an entirely separate capacity testing environment, because the environment will be as production-like as it can be.
- Although the name for this technique might not be familiar [1], the practice of canary releasing has been adopted for some time. Sometimes it is referred to as a phased rollout or an incremental rollout.
- In large, distributed scenarios, instead of using a router to decide which users will be redirected to the new version, it is also common to use different partitioning strategies. For example: if you have geographically distributed users, you can rollout the new version to a region or a specific location first; if you have multiple brands you can rollout to a single brand first, etc. Facebook chooses to use a strategy with multiple canaries, the first one being visible only to their internal employees and having all the FeatureToggles turned on so they can detect problems with new features early.
    - o
    - ![](../imgs/facebook-canary-strategy.jpg)
    - o          
- Canary releases can be used as a way to implement A/B testing due to similarities in the technical implementation. However, it is preferable to avoid conflating these two concerns: while canary releases are a good way to detect problems and regressions, A/B testing is a way to test a hypothesis using variant implementations. If you monitor business metrics to detect regressions with a canary [2], also using it for A/B testing could interfere with the results. On a more practical note, it can take days to gather enough data to demonstrate statistical significance from an A/B test, while you would want a canary rollout to complete in minutes or hours.
- One drawback of using canary releases is that you have to manage multiple versions of your software at once. You can even decide to have more than two versions running in production at the same time, however it is best to keep the number of concurrent versions to a minimum.
- Another scenario where using canary releases is hard is when you distribute software that is installed in the users' computers or mobile devices. In this case, you have less control over when the upgrade to the new version happens. If the distributed software communicates with a backend, you can use ParallelChange to support both versions and monitor which client versions are being used. Once the usage numbers fall to a certain level, you can then contract the backend to only support the new version.
- Managing database changes also requires attention when doing canary releases. Again, using ParallelChange is a technique to mitigate this problem. It allows the database to support both versions of the application during the rollout phase.    

##Spark 
- o
- ![](../imgs/streaming-arch.png)
- o
- o
- ![](../imgs/streaming-flow.png)
- o
- o
- ![](../imgs/Apache-Spark-Streaming-ecosystem-diagram.png)
- o

## ACID 
- Atomic: Everything in a transaction succeeds or the entire transaction is rolled back.
- Consistent: A transaction cannot leave the database in an inconsistent state.
- Isolated: Transactions cannot interfere with each other.
- Durable: Completed transactions persist, even when servers restart etc.

## CORS
- Cross-Origin Resource Sharing (CORS) is a mechanism that uses additional HTTP headers to tell browsers to give a web application running at one origin, 
    - access to selected resources from a different origin. 
    - A web application executes a cross-origin HTTP request when it requests a resource that has a different origin (domain, protocol, or port) from its own.
- An example of a cross-origin request: 
    - the front-end JavaScript code served from https://domain-a.com uses XMLHttpRequest to make a request for https://domain-b.com/data.json.
- For security reasons, browsers restrict cross-origin HTTP requests initiated from scripts. 
    - For example, XMLHttpRequest and the Fetch API follow the same-origin policy. 
    - This means that a web application using those APIs can only request resources from the same origin the application was loaded from, 
    - unless the response from other origins includes the right CORS headers.
- The CORS mechanism supports secure cross-origin requests and data transfers between browsers and servers. 
    - Modern browsers use CORS in a APIs such as XMLHttpRequest or Fetch to mitigate the risks of cross-origin HTTP requests.
    
## Solr:
- It's an open source search platform written in Java from Apache Lucene project
- Major features:
    - Full text search: Searching document in full text database rather than metadata
    - Hit highlighting: Term search 
    - Faceted search: Search as per faceted classification system
    - Real-time indexing
    - Dynamic clustering: Can have nodes
    - Database integration
    - NoSQL features
    - Rich Document handling. 
- Solr is widely used for search & analytic use case
- It runs as a standalone full-text search server. 
    - It uses the Lucene Java search library at its core for full-text indexing & search. 
- It has REST like HTTP/xml & JSON APIs that make it usable from programming languages. 
- Solr's external configuration allows it to be tailored to many types of application without Java coding.     