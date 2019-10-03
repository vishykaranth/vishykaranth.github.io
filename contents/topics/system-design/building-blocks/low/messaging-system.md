## Messaging System
### Background 
- log files might be a better representation of streams rather than the tables, 
- and hence we resolved to Apache Kafka. 
- Apache Kafka claims a central position in the event streaming architecture, 
- and we also learned that Apache Kafka implements a messaging system or the pub/sub semantics. 
- However, we do not know why? 
- Before we start learning Kafka architecture, it makes sense to quickly refresh the main idea of a messaging system 
- and highlight why and how Kafka implements the same notion.

### Data Integration Problem
- We cannot develop a single standalone application that does everything in an enterprise. 
- Applications can be custom designed, or they can be purchased from the third-party vendors. 
- Some of the applications may be running outside of the organization and maintained by the partners or the service providers. 
- These applications may be running on multiple computers, on different platforms, using different technologies and could be geographically dispersed. 
- We need to integrate these applications together to achieve a unified set of functionalities. 
- Data integration among the systems is not a new requirement. 
- However, when you are transforming your organization to respond to event streams, the data integration becomes more challenging.
- Real-time data integration requires a range of considerations and consequences
    - Time sensitivity of events
        - One of the critical requirements of real-time data integration is to make data available to other applications in a time-critical manner.
        - The simplest method to accomplish real-time integration is by exchanging data as frequently as possible and do it in small chunks.    
    - Decoupling of applications
        - The next requirement is to achieve decoupling of applications. 
        - Strong coupling of applications is based on some assumptions about how the other application works. 
        - These assumptions create dependencies and hurdles in evolving the apps over time. 
        - Therefore, the integration should be general enough to allow both the applications to change as needed. 
        - The integration goal should be to minimize the dependency among data producers and data consumers and yet achieve the integration needs.    
    - Data format evolution and extensibility
        - The data format is another critical problem. 
        - We can’t change existing applications to use a unified data format. 
        - We may need the flexibility to add an intermediate translator to unify the applications that insist on different data formats. 
        - The data producers may also evolve over time, 
        - and we need the flexibility to support the evolution and extensibility of the data formats as the applications, and the business need evolves.    
    - Reliability
        - Different applications may be scattered on discrete computers across geographies. 
        - Establishing a synchronous connection or making a remote procedure call may not be a reliable option as the systems may not be available all the time or the network may be temporarily unavailable. 
        - These problems can be simplified by using asynchronous communication and placing a reliable middleman to facilitate communication.
    - Scalability.
        - Scalability is obvious. 
        - We are living in the big data era, and we want our systems to be horizontally scalable. 
        - You may start with the couple hundred events a day, but as the business grows, the solution should be scalable to supports millions and trillions of events. 

### Integration Alternatives
- Remote procedure invocation
    - RMI or RPC is mainly used to trigger some action using APIs at the other application and provide the necessary data for the operation. The API is typically offered by the receiver application, and it needs to be implemented by the sender application. This approach creates a tight coupling among the applications. We already evaluated this option in the earlier chapter and realized that it fails on the decoupling and scalability parameters.
- Shared database
    - All the data producers write to a shared database, and all the data consumers read from the same database. 
        - This option fails on many parameters such as data format evolution and extensibility, scalability, and reliably supporting applications dispersed across the locations.
    - Designing a unified schema that meets the need for all the participating applications is a practically impossible task. 
        - This problem becomes more prominent if you need to integrate with packaged applications or the external applications that are not flexible enough to adapt to the shared schema design.
    - Multiple applications simultaneously reading and modifying the same data will bottleneck the database performance. 
        - When systems are distributed across various locations, accessing a single, shared database is not practical.
- File transfer
    - File transfer is the most straightforward pattern for implementing data integration. 
    - The data producer application produces a file, and all the data consumer applications are supposed to consume the data files. 
    - The most obvious example is the export-import operations. 
    - These files could be XML, JSON or maybe a simple delimited file formats such as CSV. 
    - However, this approach is obviously not going to work for real-time requirements, and they fail on the time-sensitivity parameter. 
    - Generating files and handling them on-time may get you into many other trade-offs.
- Messaging
    - The final data integration pattern is the messaging system. 
    - Apache Kafka adopted a messaging system pattern to solve the real-time data integration problem. 

### Enter the Messaging System
- A messaging system is all about sending small chunks of data from one system to other systems. 
- For our retail example, a single invoice could be considered as a message. 
- Messages could be a JSON record or something similar. 
- There are two approaches to messaging.
    - Point to Point
        - A point-to-point (PTP) messaging system is built on the concept of message queues, senders, and receivers. 
        - Each message is addressed to a specific queue, and receiving clients extract messages from their designated queues. 
        - These queues retain all messages sent to them until the messages are consumed or expire. 
        - PTP messaging has the following characteristics.
            - Each message has only one consumer.
            - Sender and receiver are not dependent on each other.
            - The receiver acknowledges the successful receipt of a message.
        - As the name suggests, PTP is more suitable for the one-to-one scenario. 
        - Apache Kafka rejected PTP approach because they wanted to address many-to-many requirement.    
    - Publish/Subscribe or pub/sub
        - A publish/subscribe (pub/sub) messaging system is built on the concept of publishers, broker, topic, and subscribers. 
        - Let’s try to understand the components of a pub/sub messaging system.
            - Publisher : 
                - Any application that sends data should act as a publisher. 
                - For our retail example, the POS application works as a publisher, and it sends the invoices for other apps. 
                - People use different names such as data publisher, data producer, and sender, however, it means the same thing – a publisher.
            - Subscriber
                - An application that reads data sent by the publisher is a subscriber. 
                - For our retail example, the services such as shipment and loyalty are subscribers. 
                - They read invoices that are posted by the POS systems. 
                - You might notice people referring to them as data consumers or receivers; however, all that means the same thing – a subscriber.
            - Broker
                - The broker is at the heart of the pub/sub messaging system. 
                    - The broker is responsible for receiving messages from the publishers, storing them into a log file, and sending the messages to the subscribers. 
                    - The broker acts as a middleman between publishers and the subscribers.
                - Apache Kafka is a message broker, and it offers two sets of APIs.
                    - Producer API
                        - Any application that wants to send a message should use producer API to send the data to Apache Kafka. 
                        - Kafka receives the message, sends acknowledgment and persists the data into a log file.                    
                    - Consumer API
                        - When a consumer application wants to read the message, they use consumer API to read the messages from the broker.


 

Topic
The topic is the mechanism to categorize the messages. If you are coming from the database world, you can think of the topic as a table name. The producer always writes the message to a topic, and the consumer reads it from the topic. The broker creates a log file for each topic. When a producer sends a message, it specifies the topic name for the message and accordingly the broker persists the message in the corresponding log file. When a consumer wants to read the message, they consume messages from the specified topic.

Kafka in the data ecosystem
Apache Kafka can become a circulatory system of your data ecosystem that carries messages to various members of the infrastructure. It occupies a central place in your real-time data integration infrastructure.

Apache Kafka in Data Ecosystem
Fig.1 - Apache Kafka in Data Ecosystem
The data producers can send data as messages, and they can send it to Kafka broker as quickly as the business event occurs. Data consumers can consume the messages from the broker as soon as they arrive at the broker. With careful design, the messages can reach from producers to consumers in milliseconds.
The producers and consumers are completely decoupled, and they do not need a tight coupling or direct connections. They always interact with the Kafka broker using a consistent interface. Producers do not need to be concerned about who is using the data, and they just send the data once without caring about how many consumers would be reading the data. Producers and consumers can be added, removed, and modified as the business case evolves.
When coupled with a system to provide message schema service and a connector service, the producer and consumers do not need to agree on a unified schema. The producer application can expose the data in the format they desire. You can always place a connector that handles the translation process for the consumer. With the flexibility to put the translator in between, you do not need to ensure that the consumer understands the producer message format. Message schema service and the connector framework are add-on features of Apache Kafka infrastructure. We will cover them in a separate chapter.
The above discussion qualifies pub/sub messaging systems for the first three decision criteria that we listed in the earlier section. However, we still need to ensure reliability and scalability. These two features are offered by Kafka architecture and design. Apache Kafka is not a traditional pub/sub messaging system. It is a distributed, fault tolerant and highly scalable system that is designed to provide the backbone infrastructure for real-time streaming requirements. We will cover the reliability and scalability in the next chapter.


 

Summary
In this lesson, we listed out some critical requirements for an acceptable data integration solution. Then I talked about four high-level alternatives and evaluated all of them against the decision criteria. Finally, I introduced you to the messaging systems and explained the point-to-point as well as pub/sub semantics. Then we went ahead and evaluated the pub/sub messaging systems against our requirement for the real-time data integration solution. We concluded this chapter by realizing that the pub/sub could be an excellent alternative for real-time data integration. In the next chapter, we will talk about Kafka architecture and design to understand how Kafka provides reliability and scalability to the real-time streaming platform.