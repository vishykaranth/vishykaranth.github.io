##REST 101
###Resource Oriented
- A resource is something that can be stored on a computer and represented as a stream of bits.
- Resource-Oriented refers to modelling each entities as Resources which can be accessed by at least one identifier.
###Representation
- Some data about the current state of a resource.
###Addressability
- Each resource should be accessible through at least one identifier.
###Statelessness
- The service never relies on information from previous requests.
- There are 2 types of states in REST.
    - Resource state
        - The representation of a resource which is stored on a server.
        - This state is same for all the clients.    
    - Application state
        - The information that are stored on respective client.
        - Each client may have different state.
        - This information or state is provided to the service while making a request.
        - Note:Don’t get confused with the in-memory state of service.
        - This state will be created whenever there is a client request and flushed out after sending the response back to client.    
###Scalability
- Capability to handle large number of requests.
    - X-Axis scaling : Running multiple copies of entire application.
    - Y-Axis scaling : Running multiple copies of a particular service which has huge number of requests at a particular instant of time.
###Connectedness
- Sometimes representation of a resource is not just a stream of bits,but it may contain links to other resources.
###Uniform interface
- Deals with how a client talk to a service and understands what to tell the service.
- Also the service should be able to understand what clients wants to say.