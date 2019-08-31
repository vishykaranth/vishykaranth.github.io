//https://github.com/hph29/wiki/wiki/%5BEbook%5D-Confluent-Design-Event-Driven-System
## Design Event-driven System
A typical streaming application that ingests data from mobile devices into Kafka, processes it in a streaming layer, and then pushes the result to a serving layer where it can be queried

-  Kafka vs. Asynchronous RESTful:
    - The difference is the presence of a broker.
    - A broker is a separate piece of infrastructure that broadcasts messages to any programs that are interested in them, as well as storing them for as long as is needed
    - It’s perfect for streaming or fire-and-forget messaging.
    - Kafka is a mechanism for programs to exchange information, but its home ground is event-based communication, where events are business facts that have value to more than one service and are worth keeping around.

- Kafka vs. ESB (Enterprise Service Bus)
    - ESBs focus on the integration of legacy and off-the-shelf systems, using an ephemeral and comparably low throughput messaging layer, which encourages request-response protocols.
    - Kafka, however, is a streaming platform, and as such puts emphasis on high throughput events and stream processing.
    - A Kafka cluster is a distributed system at heart, providing high availability, storage, and linear scale-out.
    - Streaming encourages services to retain control, particularly of their data, rather than providing orchestration from a single, cen‐ tral team or platform

- The core mantra of event-driven services: 
    - Centralize an immutable stream of facts. 
    - Decentralize the freedom to act, adapt, and change.

- Kafka vs. Database: 
    - Kafka is designed to move data, operating on that data as it does so. It’s about real-time processing first, long-term storage second.

- What is Kafka: 
    - Kafka is a streaming platform. At its core sits a cluster of Kafka brokers
    - You can interact with the cluster through a wide range of client APIs in Go, Scala, Python, REST, and more.
    - There are two APIs for stream processing: Kafka Streams and KSQL. These are database engines for data in flight, allowing users to filter streams, join them together, aggregate, store state, and run arbitrary functions over the evolving stateful dataflow.
    - The third API is Connect. This has a whole ecosystem of connectors that interface with different types of database or other endpoints, both to pull data from and push data to Kafka.
    - Finally there is a suite of utilities—such as Replicator and Mirror Maker, which tie disparate clusters together, and the Schema Registry, which validates and manages schemas—applied to messages passed through Kafka and a number of other tools in the Confluent platform.

- Service-based architectures, like microservices or SOA, are commonly built with synchronous request-response protocols. This approach is very natural. It is, after all, the way we write programs: we make calls to other code modules, await a response, and continue. It also fits closely with a lot of use cases we see each day: front-facing websites where users hit buttons and expect things to happen, then return.

- As the number of services grows gradually, the web of synchronous interactions grows with them.

- One is to ensure each individual service has a significantly higher SLA than your system as a whole.

- An alternative is to simply break down the synchronous ties that bind services together using (a) asynchronicity and (b) a message broker as an intermediary.

- There are three distinct ways that programs can interact over a network: commands, events, and queries.

- Commands are actions—requests for some operation to be performed by another service, something that will change the state of the system. 
    - Commands execute synchronously and typically indicate completion, although they may also include a result.
    - On operations that must complete synchronously, or when using orchestration or a process manager. Consider restricting the use of commands to inside a bounded context.
- Events are both a fact and a notification. They represent something that happened in the real world but include no expectation of any future action. They travel in only one direction and expect no response (sometimes called “fire and forget”), but one may be “synthesized” from a subsequent event.
    - When to use: when loose coupling is important (e.g., in multiteam systems), where the event stream is useful to more than one service, or where data must be replicated from one application to another. Events also lend themselves to concurrent execution.
        - Behavior/State change	Includes a response
        - Command	Requested to happen	Maybe
        - Event	Just happened	Never
        - Query	None	Always
- The beauty of events is they wear two hats:
    - a notification hat that triggers services into action,
    - but also a replication hat that copies data from one service to another.
    - From a services perspective, events lead to less coupling than commands and queries.
    - Loose coupling is a desirable property where interactions cross deployment boundaries, as services with fewer dependencies are easier to change.
- Messaging helps us build loosely coupled services because it moves pure data from a highly coupled place (the source) and puts it into a loosely coupled place (the subscriber).
- Any operations that need to be performed on that data are not done at source, but rather in each subscriber, and messaging technologies like Kafka take most of the operational stability/performance issues off the table.
- By using event-driven architecture,
    - Better isolation and autonomy; Isolation is required for autonomy. Keeping the data needed to drive queries isolated and local means it stays under the service’s control.
    - Faster data access; Local data is typically faster to access. This is particularly true when data from different services needs to be combined, or where the query spans geographies.
    - Where the data needs to be available offline; In the case of a mobile device, ship, plane, train, or the like, replicating the dataset provides a mechanism for moving and re-synchronizing when connected.
- By using REST/PRC approach,
    - Simplicity; It’s simpler to implement, as there are fewer moving parts and no state to manage.
    - Singleton; State lives in only one place (inevitable caching aside!), meaning a value can be changed there and all users see it immediately. This is important for use cases that require synchronicity—for example, reading a previously updated account balance.
    - Centralized control; Command-and-control workflows can be used to centralize business processes in a single controlling service. This makes it easier to reason about.
- Event Collaboration: Multiple components work together by communicating with each other by sending events when their internal state changes.
- The lack of any one point of central control means systems like these are often termed choreographies: each service handles some subset of state transitions, which, when put together, describe the whole business process.
- This can be contrasted with orchestration, where a single process commands and controls the whole workflow from one place for example, via a process manager.5 A process manager is implemented with request-response.
- _Event Sourcing_ is just the observation that events (i.e., state changes) are a core element of any system. So, if they are stored, immutably, in the order they were created in, the resulting event log provides a comprehensive audit of exactly what the system did. What’s more, we can always rederive the current state of the system by rewinding the log and replaying the events in order.
- Command–query separation (CQS) is a principle of imperative computer programming.: It states that every method should either be a command that performs an action, or a query that returns data to the caller, but not both.
- In databases, change data capture (CDC) is a set of software design patterns used to determine (and track) the data that has changed so that action can be taken using the changed data. Also, Change data capture (CDC) is an approach to data integration that is based on the identification, capture and delivery of the changes made to enterprise data sources.
- Event Sourcing and CQRS make events first-class citizens. This allows systems to relate the data that lives inside a service directly to the data it shares with others.
- Services encapsulate the data they hold to reduce coupling and aid reuse; databases amplify the data they hold to provide greater utility to their user.
- Data services share needs to be treated differently from the data they hold internally. Data on the outside is hard to change, because many programs depend upon it. But, for this very reason, data on the outside is the most important data of all.
- A second important insight is that service teams need to adopt an openly outward-facing role: one designed to serve, and be an integral part of, the wider ecosystem.
- The problem is that none of the approaches available today—service interfaces, messaging, or a shared database—provide a good solution for dealing with this transition, for the following reasons:
    - Service interfaces form tight point-to-point couplings, make it hard to share data at any level of scale, and leave the unanswered question: how do you join the many islands of state back together?
    - Shared databases concentrate use cases into a single place, and this stifles progress.
    - Messaging moves data from a tightly coupled place (the originating service) to a loosely coupled place (the service that is using the data). This means datasets can be brought together, enriched, and manipulated as required. Moving data locally typically improves performance, as well as decoupling sender and receiver. Unfortunately, messaging systems provide no historical reference, which means it’s harder to bootstrap new applications, and this can lead to data quality issues over time
- A better solution is to use a replayable log like Kafka. This works like a kind of event store: part messaging system, part database. Messaging turns highly coupled, shared datasets (data on the outside) into data a service can own and control (data on the inside). Replayable logs go a step further by adding a central reference.
- The data dichotomy highlights this question, underlining the tension between the need for services to stay decoupled and their need to control, enrich, and combine data in their own time.
- This leads to three core conclusions: (1) as architectures grow and systems become more data-centric, moving datasets from service to service becomes an inevitable part of how systems evolve; (2) data on the outside—the data services share—becomes an important entity in its own right; (3) sharing a database is not a sensible solution to data on the outside, but sharing a replayable log better balances these concerns, as it can hold datasets long-term, and it facilitates eventdriven programming, reacting to the now.
- A streaming engine and a replayable log have the core components of a database.
- Stream processing segregates responsibility for data storage away from the mechanism used to query it.
- Kafka streaming data integration. more specifically:
    - The log makes data available centrally as a shared source of truth but with the simplest possible contract. This keeps applications loosely coupled.
    - Query functionality is not shared; it is private to each service, allowing teams to move quickly by retaining control of the datasets they use.
- The “database inside out” idea is important because a replayable log, combined with a set of views, is a far more malleable entity than a single shared database.
- Stream processing can be viewed as a database turned inside out, or unbundled. In this analogy, responsibility for data storage (the log) is segregated from the mechanism used to query it (the Stream Processing API).
- There are two main drivers for pushing data to code in this way: • As a performance optimization, by making data local • To decouple the data in an organization, but keep it close to a single shared source of truth
- Lean data is a simple idea: rather than collecting and curating large datasets, applications carefully select small, lean ones—just the data they need at a point in time—which are pushed from a central event store into caches, or stores they control.
- One interesting consequence of using event streams as a source of truth is that any data you extract from the log doesn’t need to be stored reliably.
- Ephemeral means "lasting a very short time." In computing, an ephemeral object can be very useful: for instance, in cryptography an ephemeral key is a cryptographic key used only once and discarded very quickly.
- For example, if you were importing customer information from a messaging system into a database when the customer message schema undergoes a breaking change, you would typically craft a database script to migrate the data forward, then subscribe to the new topic of messages. If using Kafka to hold datasets in full, instead of performing a schema migration, you can simply drop and regenerate the view.
- Lean practices encourage us to stay close to these shared facts, a process where we manufacture: Lean Data views that are specifically optimized to the problem space at hand.
- Event-driven systems leverage asynchronous broadcast, deliberately removing the need for global state and avoiding synchronous execution.
- There are two consequences of eventual consistency in this context:
    - Timeliness If two services process the same event stream, they will process them at different rates, so one might lag behind the other. If a business operation consults both services for any reason, this could lead to an inconsistency.
        - update view and sending confirmation email when buyer place a order, if this is not acceptable, can always add serial execution back in. The call to the orders service might block until the view is updated
        - Alternatively, we might have the orders service raise a View Updated event, used to trigger the email service after the view has been updated.
    - Collisions If different services make changes to the same entity in the same event stream, if that data is later coalesced—say in a database—some changes might be lost.
        - Collisions occur if two services update the same entity at the same time.
        - Tax and services fee services run together to operate a transaction, ending up with a trade with tax added and same trade with service fee added
        - This means that, to get the correct order, with both services fee and sales tax applied, we would have to merge these two messages.
- A good compromise for large business systems is to keep the lack of timeliness (which allows us to have lots of replicas of the same state, available read-only) but remove the opportunity for collisions altogether (by disallowing concurrent mutations). We do this by allocating a single writer to each type of data (topic) or alternatively to each state transition.
A useful way to generify these ideas is to isolate consistency concerns into owning services using the single writer principle
Responsibility for propagating events of a specific type is assigned to a single service—a single writer.
It allows versioning (e.g., applying a version number) and consistency checks
It isolates the logic for evolving each business entity, in time, to a single service, making it easier to reason about and to change
It dedicates ownership of a dataset to a single team, allowing that team to specialize.
Instead of sharing a global consistency model (e.g., via a database), we use the single writer principle to create local points of consistency that are connected via the event stream. There are a couple of variants on this pattern.
Command Topic: A common variant on this pattern uses two topics per entity, often named Command and Entity. This is logically identical to the base pattern, but the Command topic can be written to by any process and is used only for the initiating event. The Entity topic can be written to only by the owning service: the single writer.
Service	Orders service	OrdersTopic
Event	OrderRequest(ed)	OrderValidated, OrderCompleted
Writer	Any service	Orders service
Single Writer Per Transition A less stringent variant of the single writer principle involves services owning individual transitions rather than all transitions in a topic. So, for example, the payment service might not use a Payment topic at all. It might simply add extra payment information to the existing order message (so there would be a Payment section of the Order schema). The payment service then owns just that one transition and the orders service owns the others.
Service	Orders service	Payment service
Topic	OrdersTopic	OrdersTopic
Writable transition	OrderRequested->OrderValidated PaymentReceived->OrderConfirmed	OrderValidated->PaymentReceived
Atomicity with Transactions Kafka provides a transactions feature with two guarantees: • Messages sent to different topics, within a transaction, will either all be written or none at all. • Messages sent to a single topic, in a transaction, will never be subject to duplicates, even on failure.
But transactions provide a very interesting and powerful feature to Kafka Streams: they join writes to state stores and writes to output topics together, atomically.
The single writer principle and other related patterns discussed in this chapter are exactly that: patterns, which are useful in common cases, but don’t provide general solutions. There will always be exceptions, particularly in environments where the implementation is constrained, for example, by legacy systems.
Idempotence is required in the broker to ensure duplicates cannot be created in the log. Idempotence, in this context, is just deduplication. Each producer is given an identifier, and each message is given a sequence number. The combination of the two uniquely defines each batch of messages sent. The broker uses this unique sequence number to work out if a message is already in the log and dis‐ cards it if it is. This is a significantly more efficient approach than storing every key you’ve ever seen in a database.
Kafka can be used to store data in the log, with the most common means being a state store (a disk-resident hash table, held inside the API, and backed by a Kafka topic) in Kafka Streams. As a state store gets its dura‐ bility from a Kafka topic, we can use transactions to tie writes to the state store and writes to other output topics together. This turns out to be an extremely powerful pattern because it mimics the tying of messaging and databases together atomically, something that traditionally required painfully slow proto‐ cols like XA.
How Kafka implement Transaction messages: Two messages were written to two different topics atomically. One message goes to the Deposits topic, the other to the committed_offsets topic. Begin markers are sent down both. We then send our messages. Finally, when we’re done, we flush each topic with a Commit (or Abort) marker, which con‐ cludes the transaction.
To ensure each transaction is atomic, sending the Commit markers involves the use of a transaction coordinator. There will be many of these spread throughout the cluster, so there is no single point of failure, but each transaction uses just one. The transaction coordinator is the ultimate arbiter that marks a transaction com‐ mitted atomically, and maintains a transaction log to back this up
Transactions affect the way we build services in a number of specific ways:
They take idempotence right off the table for services interconnected with Kafka. So when we build services that follow the pattern “read, process, (save), send,” we don’t need to worry about deduplicating inputs or constructing keys for outputs.
We no longer need to worry about ensuring there are appropriate unique keys on the messages we send. This typically applies less to topics containing business events, which often have good keys already. But it’s useful when we’re managing derivative/intermediary data—for example, when we’re remapping events, creating aggregate events, or using the Streams API.
Where Kafka is used for persistence, we can wrap both messages we send to other services and state we need internally in a single transaction that will commit or fail. This makes it easier to build simple stateful apps and services.