## REST 101

### Resource Oriented
- A resource is something that can be stored on a computer and represented as a stream of bits.
- Resource-Oriented refers to modelling each entities as Resources which can be accessed by at least one identifier.
### Representation
- Some data about the current state of a resource.
### Addressability
- Each resource should be accessible through at least one identifier.
### Statelessness
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
### Scalability
- Capability to handle large number of requests.
    - X-Axis scaling : Running multiple copies of entire application.
    - Y-Axis scaling : Running multiple copies of a particular service which has huge number of requests at a particular instant of time.
### Connectedness
- Sometimes representation of a resource is not just a stream of bits,but it may contain links to other resources.
### Uniform interface
- Deals with how a client talk to a service and understands what to tell the service.
- Also the service should be able to understand what clients wants to say.

### Guidelines for REST using HTTP
#### Idempotent HTTP methods
- An HTTP method is said to be idempotent if the server provides the same representation of a resource irrespective of the number of times user request it.
- Example: An image of a cat is stored at server.
    - If the user GETs this image 100 times continuously, the server will serve the same representation 100 times.
    - If the user replace this image with a new image 100 times, it is same as the first replace.
- Non-idempotent method:
    - But what if there is an image counter resource at server side which gets incremented during an HTTP request. 
    - So in this scenario, if the user requests 100 times, the counter is incremented by 100.

#### GET
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

#### POST
- To create a new resource whose URI is going to be determined by server.
- It is not idempotent.

#### PUT
- To replace the current representation of an existing resource with a new representation.
- If client knows or decides the URI to create a new resource under that URI,then use PUT.
- It is idempotent.

#### PUT Versus POST
- You can expose the creation of new resources through PUT, POST or both. 
    - But a client can only use PUT to create resources when it can calculate the final URI of the new resource.
    - Example: In Amazon’s S3 service, the URI path to the Bucket is /{Bucket-name}. 
    - Since the client chooses the Bucket name, PUT is used.
- On the other hand, the URI to a resource which is stored in a table may look like /{database-table-name}/{database-ID}.
- The name of the database table is known in advance, but ID is created run time. 
    - In such scenario, POST is used to send request to a a “Factory” resource located at /{database-table-name}.
    - The server chooses a URI for the new resource.

#### URI naming
- Use path variables to separate elements of a hierarchy. It can be imagined as a directed graph.
- Use punctuation characters to separate multiple pieces of data at the same level of a hierarchy.
- Use Query variables only to suggest arguments being plugged into an algorithm.
- URIs are supposed to designate resources,not operations on the resources.
    - So the expected values on the URI are Nouns.But sometimes,some operation can themselves be a Resource.
    - In such scenario it is ok to have URI with /{do-operation}
    
#### HTTP Status codes
- Whenever there is an error condition, send an HTTP response code in the range 3xx,4xx or 5xx and provide supplementary data in HTTP headers.
    - If the server provides a response body during error condition,then it will be a document describing the error condition,not a representation of the requested resource.
    - 404 “Not found” is used when the user try to access a resource which doesn’t exist.It is helpful to send some description about the error in the body as it helps in debugging,but usually only status code is send in production environment.
    - 303 “See other” can be used when user has mistyped some URI and the server can somehow figure out that he/she is trying to use some valid URI.
    - 400 “Bad Request” can be used when user has typed a logically impossible values in the URI but it matches the URI template server has defined.
        - For example: /latitude/500,-181, 500,-181 is not logically possible and there is no resource.
    - 503 “Service unavailable” can be used when the server is unable to fulfill that particular request.
    - 500 “Internal Server Error” can be used to express some condition occurred due to corrupted data,hardware failure or a software bug.
- Happy codes:
    - 200 “OK” is used when server sends the expected result the client wanted.
    - 201 “Accepted” is used to acknowledge that the job has been created at server side and come back later to know the status of the job.(Asynchronous)
    
#### Asynchronous,Batch,Transaction using HTTP REST
##### Relationship between Resources:
- Suppose Alice and Bob are resources.One day they got married.A client can PUT to Alice URI that she got married to Bob and can PUT to Bob’s URI that he got married to Alice.In short,2 PUT request on two different URIs.
- The best way is to consider the relationship itself as resource.A client can send PUT request to Marriage URI to declare two people married.
##### Asynchronous Operations:
- HTTP has a synchronous request-response model.
    - The client opens a socket ,makes a request to the server and keeps the socket open until the server has sent the response.
    - If the client doesn’t care about the response, it can close the socket early.
    - But if it wants a response, then it leave the socket open until the server sends the response.
- The problem is not all operations can be completed in the time we expect.
    - Some operations may take hours or days.
    - An HTTP request should be timed out after some kind of inactivity. 
    - So how to find a RESTful solution for such time consuming operation?
- The solution is that the operation be split into two or more synchronous requests.
    - The first request spawns the operation and subsequent requests let the client learn about the status of the operation.
    - Example: Imagine that we have a web service that handles a queue requests.
        - The client makes its service requests normally, possibly without any knowledge that the request will be handed asynchronously.
        - It sends request like this one:
        ~~~java
            Post /queue
            Host :jobs.example.com
            Give me the square root of 123223422322233
        ~~~
- The server accepts the request, creates a new job and puts it at the end of the queue.
    - It will take a long time for the new job to be completed.
    - Instead of keeping the client waiting until the job finally runs, the server sends this response right away:
    ~~~java
    202 Accepted
    location:http://jobs.example.com/queue/job345
    ~~~
- The client will use this URI to check the status of the job.
##### Batch Operations:
- Sometimes clients need to operate on more than one resource at once.
- What if he/she wants to modify a set of resources at once?
- It was stated earlier that each URI should denote a resource or set of resources.
    - If user wants to DELETE a set of resources at once then he/she will send DELETE request to that URI which represents the set.
    - But what the status code? what if the server fails to delete one of the resource in hat set?
- One solution is 202 “Accepted” and send client a link which he/she can use later to check the status.
- Another solution is 207 “Multi-Status”,which tells the client to look into the response body for a list of status codes.            