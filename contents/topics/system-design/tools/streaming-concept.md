## Streaming Concepts
### Background 
- the business activities can be modelled as a stream of events. 
- Likewise, there is a lot of hype around “machine-generated data” and “Internet of things.” 
- These buzzwords may have a different meaning for different people, but a considerable part of these areas is about the collection and processing of big data streams. 
- Like any other domain, stream processing has its own set of challenges and concepts. 
- challenges and concepts that you will be dealing with while developing your streaming solutions.

### Detaching Fallacies
- The popularity and increasing demand for stream processing is a natural progression of Bigdata evolution. 
- Most of the stream processing platforms are built by taking capabilities from the big-data batch processing world and making them available for a low-latency domain. 
- This approach created some misconception around the notion of stream processing. In this section, I will try to highlight the central fallacies of stream processing.

### Stream Processing Vs Analytics
- People often take stream processing synonymous with real-time analytics. 
- Real-time analytics is one important use case of stream processing. 
- However, it doesn’t always mean analytics. 
- There are many more applications of real-time stream processing. 
- More often, your stream processing application would implement core functions in the business rather than computing analytics about the business. 
- Analytics may be an individual service along with many other services on your streaming platform. 
- Stream processing is a backbone infrastructure and a set of services that coordinate with analytics function. 
- It is more like the glue that binds your platform to deliver the business processes that may also include analytics.

### Stream Processing Vs ETL
- Some people take the stream processing as an ETL pipeline that works in real-time. 
- That might be a good starting point on stream processing. 
- However, stream processing applications require addressing needs that are very different from the ETL domain of conventional big-data processing. 
- Stream processing application is more like microservices rather than a scheduled job. 
- Like a typical microservice, stream processing applications might not be processing requests over HTTP, but they operate on asynchronous event streams over a pull mechanism. You can take the stream processing applications as an application programming model for asynchronous services rather than an ETL framework.

### Stream Processing Vs Cluster Computing
- Cluster Computing
    - Cluster computing or the distributed computing is all about sharing the workload over a pool of resources in the cluster of computers. 
    - This is often implemented using a framework that is responsible for providing the fault tolerance, scalability, and many other orchestration capabilities. 
    - These cluster management frameworks manage the resources in the form of containers which is nothing but a logical bundle of cluster resources such as memory, CPU, networking and in many cases operating system image as well. 
    - This area is going under a separate evolution, and we can see a further decoupling of responsibilities for greater flexibility. 
    - Docker containers have decoupled the container configuration and packaging from the cluster management. 
    - The Kubernetes is trying to solve the problem of resource allocation and placement of containers in a fault tolerant and scalable cluster.
- Stream Processing    
    - A stream processing application has a different domain of problems and challenges while dealing with the stream of events in the real-time. 
    - They have nothing to do with the cluster management and resource allocation, 
    - and in fact, you would want to have the flexibility to take advantage of the separate developments that are happening in the cluster computing space.
    - Many of the big-data processing frameworks such as Apache Spark assumes the responsibility of packaging the dependencies, 
    - serialize your code and send it to the workers over the network. 
    - This process has an inherent delay to start the execution, 
    - and it also imposes a restriction on packaging and deployment flexibility that in turn takes away a bunch of advantages of CI/CD and few other things.
- To summarize this discussion, 
    - when you are designing a stream processing solution, 
    - you may not want to get trapped into the cluster management requirements. 
    - You stream processing application should not have a dependency on the cluster management frameworks. 
    - In fact, the cluster management platform should be entirely optional for your streaming application as well as you should have the flexibility to take advantage of any of such mature platforms.

### Stream Processing vs Batch Processing
- The first and most obvious thing that you would notice about the stream processing is that you are going to deal with an unbounded, ever growing, infinite dataset that is continuously flowing to your system.
- In contrast, the batch processing is dealing with a bounded, fixed and finite data set that has already landed at your system in the past. 
- If you carefully think about the bounded dataset, it is nothing but a small subset of the unbounded dataset that is chopped into a smaller set from the stream.
- The approach taken up by a micro-batch processing system is a bottom-up approach where they first solved a subset of the problem by processing smaller batches and tried to enhance and extend the batch method to solve a much bigger problem of dealing with infinite streams.
- However, the systems that are initially designed for the stream processing in mind, 
    - they would have the flexibility to take a radical approach to solve a bigger problem, 
    - and then use the same method to deal with the smaller batches of bounded data, 
    - that are in fact a subset of the problem that the stream processing system already solved. 
    - The point is straight. 
    - A well designed, efficient stream processing system will eventually eliminate the need for the batch processing systems.
### Attaching difficulties
- Like any other technical domain, stream processing also poses a unique set of challenges. 
- Before we start working with streams, it is critical to have some good sense of the difficulties that you are going to handle while dealing with the event streams. 
- These new concepts are instrumental for the stream processing solution design.
    - Time Domain
    - One of the capabilities of the stream processing system is the power to extract value out of the time-sensitive events before the value vanishes. For example, healthcare systems want to send an alert to nurses as soon as possible. With time, the value of such alarm disappears. This time sensitivity is not only associated with the life-critical systems. Time makes sense in many other use cases. For example, your application is processing a stream of ticket booking events.
    - Event stream of online reservation
    - Fig.1 - Event stream of online reservation
    - o
    - ![](../../../misc-reference/interview/imgs/kafka/event-stream-of-online-reservation.jpg)
    - o         
    - As the event is received to your application, you want to schedule a reminder that should be sent a couple of hours before the show. However, such a reminder doesn’t make sense If the ticket is booked fifteen minutes before the show, and hence you want to filter out those tickets from flowing to the scheduler. The point is straight. The timestamping of the event is critical for many stream processing use cases, and hence you may have to attach a timestamp with your events.

### Event-time vs Processing-time
For a stream processing system, we often care about two types of timestamps.

Event time
Processing time
Event time is the time that an event happened in the real world. For example, the time when a ticket was booked by an online ticket booking platform. Another example would be the time when the search button is clicked by the user to search some product on your eCommerce website.

Event time vs Processing time
Fig.2 - Event time vs Processing time
Processing time is the time that the event is observed by the machine that is processing it. For example, the time when the ticket booking event is received at your stream processing system that takes a decision to schedule an alert.
Both notions of time are useful depending on the application. For example, a ticket is booked at 10.30 AM for the show that starts at 11.00 AM. For some strange reasons, the ticket booking event reaches your stream processing application at 11.05 AM. You may not want to send the reminder notification now. The show should have already started. In this case, the processing time makes more sense over the event time.
On the other hand, if you are computing a search pattern over time to learn what products are being searched on your eCommerce website in the morning, versus afternoon, versus evening. For such computations, the event time makes more sense.
In an ideal world, the event time and the processing time should be same assuming it takes no time to reach the machine that processes the event. However, in a real world, the processing time would be later than the event time due to the network latency, resource sharing and a variety of other reasons.
These differences in event time and processing time may be as small as few milliseconds to many hours or even days. For example, you are collecting usage patterns for some mobile application that works online as well as offline. This data can be transmitted to your servers only when the mobile device has connectivity. If the user goes to an area where he loses the connectivity for a few hours, you may get those events after hours. If the user switches off the mobile data for a couple of days, this delay may be as significant as a few days.

Time Window
Many of the standard data processing operation appears to be time agnostic such as filter, join and grouping. Let’s assume that you are processing web traffic log entries. You want to group the records by the country of origin and count them. For this simple requirement, you may not have to consider event time or the processing time. However, the number of users by country may not be a useful metric because it keeps increasing forever. Breaking the ever-growing count in temporal boundaries should make more sense. For example, the number of users per hour or per minute for a given country would be more meaningful. The point is straightforward. Some of the requirements would need you to slice the time domain in precise time windows. You would want to collect the events generated or received within the window and perform some analysis on the whole set.
Your time window may be of two types: tumbling (fixed) or sliding.


 

Tumbling Window
Tumbling window is fixed and non-overlapping time window. To visualize the notion of tumbling window, let’s consider an input stream as shown below.

Tumbling Window
Fig.3 - Tumbling Window
The first row represents the time. Each number in the second row represents a data point. The next line groups the data by a tumbling window of one minute. Finally, we take a sum of the data points for each tumbling window.

Sliding Window
Sliding windows are fixed length, but they slide over the period. Let’s consider the same input stream that we used in the tumbling window and see how a sliding window gives you a different perspective of the time.

Sliding Window
Fig.4 - Sliding Window
The first row represents the time. Each number in the second row represents a data point. The next line groups the data by a window of one minute that slides every thirty seconds. Finally, we take a sum of the data points for each sliding window.
Now let’s talk about how these two windows are different. While using a tumbling window, you would buffer the data for a minute and then take a sum of all the data points. In this case, every computation has a delay of one minute. However, the sliding window updates the sum every thirty seconds for the most recent one minute. You would eventually get the same result as the fixed window, but you also get an early sense by an intermediate result after every thirty seconds.


 

Watermarks & Triggers
We learned that you may want to slice up your event stream into time-based windows and aggregate your results over time such as per second or per minute. However, the windowing introduces a new problem of latecomers. The stragglers can jeopardize the correctness of your results.
Let’s assume that you are you are monitoring traffic by computing the number of vehicles pass by every minute.
You started at 9.00.00 AM and wanted to sum up the number of cars that passed by between 9.00.00 and 9.01.00. However, the event takes five seconds to reach your application. So, the event that generated at 9.00.00 will arrive at your system at 9.00.05, and similarly, the event created at 9.00.59 will reach to your system by 9.01.04.

Notion of Watermark in streaming
Fig.5 - Notion of Watermark in streaming
So, to compute the correct results, you should wait until 9.01.05 to ensure that all events that are generated between 9.00 and 9.01 successfully reached your application. If you calculate your sum before that time, your output is incorrect.
The notion of waiting for the stragglers is known as watermarks or the triggers. Different stream processing system handles it differently. However, the idea is termed as the watermark. Watermarks are a feature of streaming systems that allow you to specify how late they expect to see data in event time. The triggers in this respect are the mechanism to end the waiting of the new events and trigger the computation. Watermarks and triggers are essentially an approach to handle stragglers.
Handling late record is one of the most complex problems of the stream processing system for the fact that you can’t accurately predict the delay in most of the cases. It is tough to guarantee that your application has received all the data that are generated before specific event time, and you do not have anything hanging somewhere due to some unknown reasons. This uncertainty of stragglers is the greatest streaming challenge.

Other Window Approaches
Time is one of the most common approaches to implement the notion of a window. One of the main reasons to use the time for windowing is that the time always goes on, and the time window will eventually close for sure. Another approach is to implement a window on the count of events. In this approach, we wait for some x number of records to trigger the computation. For example, you may want to wait for 100 records to arrive and then trigger the calculations. Such notions should be implemented with extra care because in certain conditions you may be waiting forever as you never got 100 records.
Another common approach is to base your window on the notion of session. In general sense, a session is a period of activity that is preceded and followed by a period of inactivity. For example, a series of interactions of a user on a website, followed by no further response for a certain period may be termed as an end of the session. Sessions are often implement using timeouts because they typically do not have a set duration or a set number of interactions. The timeout basically specifies how long we want to wait until we believe that a session has ended.


 

Stateful Streams
A state is nothing but a persistent and durable store where the application can buffer or store some data for later reference.
Many of the stream processing use cases are stateless, and they do not need to maintain any state. For example, when you are processing one event at a time, and all you need to do is to transform the event and pass it to some other processor to take necessary action. Simple transformations and filers on individual events are the most common examples that may not need to maintain a state. However, most of the stream processing applications need to keep some state, and hence they should be categorized as stateful applications.
Your application is stateful whenever it needs to aggregate, implement a window, perform a join operation, or ensure that they can be stopped and resumed.
One of the common reasons for streaming applications to maintain a state is to avoid reprocessing of huge volumes in case of failures. You may need the ability to resume from wherever you were paused or stopped. Since a streaming application works on a continuous stream of data, and if they need to go back in time and reprocess all the data once again for some reason, they may not be able to catch up with the new data in due time and hence defy the whole purpose of stream processing.

Stream Table Duality
Stream table duality is one of the most talked topics in the stream processing domain. The idea is quite simple. However, it can take you a long way to conceptualize your stream processing solutions. The notion of stream table duality talks about the relationship between data streams and database tables, and how you can create one from the other.
When implementing stream processing solutions in practice, you need streams as well as the database tables. For example, when you are processing an eCommerce transaction as an event stream, your stream processing application may want to join the transaction with the customer information from a database table. Streams are everywhere. However, the database tables are also everywhere. The point is straight. Tables are the reality, and you can’t avoid them. Tables make perfect sense in a stream processing solution as well. So, you may want to combine the ideas from both worlds and design a practical solution.

Convert Stream to table
Fig.6 - Convert Stream to table
The relationship between stream and tables can be understood more clearly by realizing the fact that a table is nothing but an aggregated view of a stream at a given point in time. This idea is applied by the databases to create tables.
In most of the databases, transactions are recorded in a log file in chronological order. The log file is a representation of a stream as it records immutable and continuous events. The databases read this log and apply them in a sequence to materialize the table. The table represents the latest state of each unique record however the log represents a sequence of transactions as they arrive. In fact, the database is converting a stream into a table.
On the other side, A table can also be converted into a stream. The idea is implemented by most of the change data capture (CDC) tools such as Oracle golden gate and HVR. These tools monitor all the changes to the table at one end of the pipeline and stream the changes to the other end of the pipe.

Convert table to stream using CDC
Fig.7 - Convert table to stream using CDC
The point is straightforward. The data exists in two related forms: streams and tables. Both are essential constituents of a practical solution.


 

Exactly Once Processing
The correctness of your results is of prime importance in most of the cases. However, some use cases are based on approximation, and a nearly accurate result is well accepted. The argument of accuracy and approximation are intensely discussed in the stream processing domain with three notions.

At least once processing
At most one processing
Exactly once processing
At-least-once processing talks about ensuring that we do not lose any event and each record is processed at least once. However, at the same time, at-least-once will leave a possibility for duplicate events. It accepts that an event is computed more than once but guarantees that none of the events are lost.
On the other hand, at-most-once processing focuses on avoiding duplicates. It guarantees that an event should not be processed more than once. However, it is acceptable to lose some of the events. In more casual terms, it means that you can miss some events, but you will never process it twice.
At-least-once and at-most-once are the two sides of an approximation. At-least-once approximates the value to the actual or more. For example, if you are counting clicks and the accurate result is 100, then the at-least-one approach will give you an answer as 100, or something more than 100. Since you chose to implement at-least-once and eliminated the possibility of losing any event, there is no chance of getting less than 100. However, the duplicate events may increase the count to something more than 100.
On the other side, at-most-once may give you 100 or something less than that as you accepted losing some events but ensured that you do not count any event twice.
At-least-once and at-most-once are about accepting tread-off depending on what makes more sense in your use case.
Exactly-once processing is ensuring that every event is processed exactly once without losing any event, and without duplicating anything as well. Exactly-once appears to be the best and the most desirable choice. However, implementing exactly-once is hard in a practical sense. One of the main problems is the delay in arriving records at the processor. You can’t keep waiting for the event forever, and it is almost impossible to ensure that the event reaches at your processor within an acceptable timeline or before the practically feasible watermark.
Streaming Concepts| 43 Some systems allow you to achieve exactly-once processing. However, that involves implementing database like transactional features to ensure that the transaction is successfully committed exactly-once. Implementing such transactions come at the cost of additional complexity, and that is why we often resolve to at-least-once, or at-most-once schematics whenever approximation is an acceptable thing.


 

Summary
In this concluding chapter of the first part of the book, we tried to clarify some common delusions that are often instigated by experience in batch processing systems. Then we moved into discussing some of the terminologies and core concepts that would be helpful for your stream processing learning journey. Some of the central ideas of stream processing are the notion of time, windowing, and persistent state. The correctness vs. approximation is the other trade-off that we would have to deal in stream processing applications. This book will be talking about all these and many more concepts throughout the chapters. However, we wanted to set the stage to start learning stream programming.