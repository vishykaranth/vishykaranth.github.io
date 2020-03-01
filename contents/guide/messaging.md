### The Messaging Model divide
- JMS defines two distinct Messaging models : 
    - Point to Point and 
        - Point to Point (P2P or PTP) is built on the concept of message queues 
            - with publisher addressing messages to specific Queue and 
            - the receiving clients extracting messages from the queue. 
            - The Queue acts as a holding area and messages are retained in the Queue until they are consumed or expire. 
            - Queues remove the timing dependency between senders and receivers.  
            - As the name Point-to-Point implies, each message can be consumed by only one consumer, though multiple consumers can bind to a queue.
    - Publish/Subscribe. 
        - With Publish/Subscribe (PubSub), 
        - messages are addressed to Topics and 
        - multiple consumers can subscribe to and receive the messages. 
        - The messaging system takes care of distributing the messages to subscribed consumers. 
        - Except for durable subscribers, messages are not retained for inactive or slow subscribers. (More on durable subscribers later).

- As a messaging application developer, you have to pick your side: 
    - PubSub for best effort fan-out subscriber use case or P2P for consumers requiring guaranteed delivery.  
    - But what if you wanted the best of both these messaging paradigms in your application? 
    - What if some of your subscribers require delivery guarantee while others don’t and you want publishers to be unaware of this dependency?

