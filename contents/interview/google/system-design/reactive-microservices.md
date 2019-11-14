## Microservices - Reactive 

### Asynchronous Message-Passing
- Using asynchronous message-passing interactions provides reactive systems with two critical properties:
    - Elasticity—The ability to scale horizontally (scale out/in)
        - Elasticity comes from the decoupling provided by message interactions.
        - Messages sent to an address can be consumed by a set of consumers using a load-balancing strategy. 
        - When a reactive system faces a spike in load, it can spawn new instances of consumers and dispose of them afterward.
    - Resilience—The ability to handle failure and recover
        - This resilience characteristic is provided by the ability to handle failure without blocking as well as the ability to replicate components.
        - First, message interactions allow components to deal with failure locally. 
        - Thanks to the asynchronous aspect, components do not actively wait for responses, so a failure happening in one component would not impact other components. 
        - Replication is also a key ability to handle resilience. 
        - When one node-processing message fails, the message can be processed by another node registered on the same address.
        
### Circuit Breakers
- A circuit breaker is a pattern used to deal with repetitive failures. 
- It protects a micro-service from calling a failing service again and again. 
- A circuit breaker is a three-state automaton that manages an interaction. 
- It starts in a closed state in which the circuit breaker executes operations as usual. 
- If the interaction succeeds,nothing happens. If it fails, however, the circuit breaker makes a note of the failure. 
- Once the number of failures (or frequency of failures, in more sophisticated cases) exceeds a threshold, the circuit breaker switches to an open state. 
- In this state, calls to the circuit breaker fail immediately without any attempt to execute the underlying interaction. 
- Instead of executing the operation, the circuit breaker may execute a fallback, providing a default result. 
- After a configured amount of time, the circuit breaker decides that the operation has a chance of succeeding, so it goes into a half-open state. 
- In this state, the next call to the circuit breaker executes the underlying interaction. 
- Depending on the outcome of this call, the circuit breaker resets and returns to the closed state, or returns to the open state until another timeout elapses.        