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

###Guidelines for REST using HTTP
####Idempotent HTTP methods
- An HTTP method is said to be idempotent if the server provides the same representation of a resource irrespective of the number of times user request it.
- Example: An image of a cat is stored at server.
    - If the user GETs this image 100 times continuously, the server will serve the same representation 100 times.
    - If the user replace this image with a new image 100 times, it is same as the first replace.
- Non-idempotent method:
    - But what if there is an image counter resource at server side which gets incremented during an HTTP request. 
    - So in this scenario, if the user requests 100 times, the counter is incremented by 100.

####GET
- Never send payload in GET requests.
- It should be Idempotent, meaning it should serve the same representation irrespective of the number of times the client requests.
- The GET URI must be descriptive,meaningful and resource oriented.
- Conditional GET : Whenever server serves a representation, it should include a time value for Last-Modified HTTP header.
    - This time denotes the last time the data underlying the representation was changed.
    - The client can store this value of Last-Modified and use it later. 
    - So the second time the client makes a GET request for that request, it can provide that time in the If-Modified-Since request Header.
    - When the server receives this GET request,it compares the time with the latest modified resource state time.
    - If both are same, then Server sends a response code of 304 “Not Modified” and doesn’t send any data.
    - But if the the time doesn’t match, then server send 200 “OK” and provides the new representation in the response body.
    - Note : Last-Modified/If-Modified-Since and ETag/If-None-Match can be used interchangeably.
5)The server should set value for Cache-control headers to avoid caching of resources at client side or Web Gateways for indefinite period of time.    

####POST
- To create a new resource whose URI is going to be determined by server.
- It is not idempotent.

####PUT
- To replace the current representation of an existing resource with a new representation.
- If client knows or decides the URI to create a new resource under that URI,then use PUT.
- It is idempotent.