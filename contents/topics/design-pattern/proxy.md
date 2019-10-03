## Proxy
- A proxy object hides the original object and control access to it. We can use proxy when we may want to use a class that can perform as an interface to something else.
- Proxy is heavily used to implement lazy loading related usecases where we do not want to create full object until it is actually needed.
- A proxy can be used to add an additional security layer around the original object as well.

## what are different types of proxies
- Proxies are generally divided into four types –
- Remote proxy – represent a remotely lactated object. 
    - To talk with remote objects, the client need to do additional work on communication over network. 
    - A proxy object does this communication on behalf of original object and client focuses on real talk to do.
- Virtual proxy – 
    - delay the creation and initialization of expensive objects until needed, where the objects are created on demand. 
    - Hibernate created proxy entities are example of virtual proxies.
- Protection proxy – 
    - help to implement security over original object. 
    - They may check for access rights before method invocations and allow or deny access based on the conclusion.
- Smart Proxy – 
    - performs additional housekeeping work when an object is accessed by a client. 
    - An example can be to check if the real object is locked before it is accessed to ensure that no other object can change it.
##Proxy pattern vs decorator pattern
- Decorators focus on adding responsibilities. 
- Proxies focus on controlling the access to an object.

## Real world example of proxy pattern
- In hibernate, we write the code to fetch entities from the database. 
    - Hibernate returns an object which a proxy (by dynamically constructed by Hibernate by extending the domain class) to the underlying entity class. 
    - The client code is able to read the data whatever it needs to read with the proxy.
- These proxy entity classes help in implementing lazy loading scenarios where associated entities are fetched only when they are requested explicitly. 
    - It helps in improving performance of DAO operations.
- In corporate networks, internet access is guarded behind a network proxy. 
    - All network requests goes through proxy which first check the requests for allowed websites and posted data to network. 
    - If request looks suspicious, proxy block the request – else request pass through.
- In aspect oriented programming (AOP), an object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). 
    - For example, in the Spring AOP, an AOP proxy will be a JDK dynamic proxy or a CGLIB proxy.