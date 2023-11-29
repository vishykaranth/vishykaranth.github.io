## DIP 

### What is Dependency Inversion Principle (DIP) about?
- Abstraction.
- High level policies.
- Classes should depend on interfaces and abstractions instead of concrete implementations.
- Abstract away the low level details that matter less to your business domain.
- e.g. You need to store data into some persistent storage. 
    - But does the type of storage matter from the domain point of view? 
    - In other words, should your domain care about details like whether the storage is a 
        - SQL database, 
        - NoSQL, 
        - a hard drive, or 
        - some other modern cloud storage etc…? 
    - The answer is usually No. 
    - Of course, at the lower level, you need to write codes that interface with these external storages. 
    - However, at the higher level, you don’t need to care about these details. 
    - In this scenario, it’s worth to introduce a layer of abstraction to address the business need. 
    - For example, you may have a DataRepository interface in which you declare methods such as Save(data) and Get(data). 
    - These methods are just what you need to address the business problem. 
    - Then, you can have concrete implementations that implement the interface. 
    - For example, you may have a class called SQLDataRepository. 
    - In this class, you have methods that actually talk to the SQL database.

### What is DIP not about?
- DIP is different than Dependency Injection (DI) or Inversion of Control (iOC). 
    - Those are three different concepts, but they play well together.
- DIP is not about reversing the dependency between two types. 
    - For instance, if A depends on B, then DIP is not about switching the dependency such that B depends on A. 
    - That would not make much sense. 
    - Instead, it’s about breaking the dependency such that A does not have a direct dependency on B. 
    - Rather, A depends on something more abstract.
- DIP is not about making every type or class abstract, as that would not be possible. 
    - For example, string in java or C# is a concrete type. 
    - However, it’s very stable and does not change often. 
    - As such, there is not a strong reason to make string abstract.
- DI is about wiring, IoC is about direction, and DIP is about shape.

### DIP in the Wild
#### DIP related concepts
- Programming to interfaces, not implementations
- Other principles in SOLID: Open Closed Principle, Liskov Substitution Principle.

#### Clean Architecture
- Why should you adhere to DIP?
    - A class which depends directly on another concrete class is not desirable because of the increase in coupling. 
    - Any change to the depended on class may affect the dependent class.
- Reduce coupling in your system, which is a good thing.
- Make your applications more resilient to changes, as a result of not having direct dependencies on concrete classes.
- Make testing easier 
    - it’s easier to introduce seams and 
    - create mocks when your classes depend on abstracts and interfaces rather than concrete implementations
#### How to better conform to DIP?
- In essence, to follow DIP is to avoid having dependencies on concrete classes. 
- While it’s not possible to have zero dependencies on concrete classes, 
    - the idea is to isolate those dependencies from the business layer as much as possible.

#### Clean Architecture (Martin, 2017), the author suggests the following:
- Avoid referring to volatile concrete classes. 
    - Instead, refer to interfaces.
- Be careful when using inheritance, especially in a statically typed language such as Java or C#. 
    - In those languages, when you have a class which derives from a concrete class, 
    - you essentially have a direct dependency between the two classes.
- Don’t override concrete functions. 
    - Doing so mean your module may have to depend on the modules which contains the functions.

#### Additional Thoughts 
- Organizing your modules following clean architecture naturally makes your application more conformant with DIP. 
- Following clean architecture, you would have all the high level policies and domain define at the core layer. 
- The core layer would not have any dependencies on third party APIs besides the framework and system classes. 
- In fact, you would define interactions with any external system via abstracts and interfaces. 
- Not having any dependencies on third party APIs also mean that the core layer would not have any classes outside of the business domain. 
- If a method returns a result, the result should be a literal value, data transfer object, or a domain related to the business. 
- Eventually, you may need to have concrete implementations that interact with external systems. 
- Those implementations can reside at the infrastructure layer, outside of your core or domain layer.