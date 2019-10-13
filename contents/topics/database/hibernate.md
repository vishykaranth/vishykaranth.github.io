- ![](imgs/Hibernate%20load.jpg) 


## Interview Questions  
- What is Hibernate?
    - Hibernate is a powerful, high performance object/relational persistence and query service.
    - This lets the users to develop persistent classes following object-oriented principles such as association, inheritance, polymorphism, composition, and collections.
- What are the collection types in Hibernate?
    - There are five collection types in hibernate used for one-to-many relationship mappings.
        - Bag
        - Set
        - List
        - Array
        - Map
- What are different states of an entity bean?
    - An entity bean instance can exist is one of the three states.
        - Transient: 
            - When an object is never persisted or associated with any session, it’s in transient state. 
            - Transient instances may be made persistent by calling save(), persist() or saveOrUpdate(). 
            - Persistent instances may be made transient by calling delete().
        - Persistent: 
            - When an object is associated with a unique session, it’s in persistent state. 
            - Any instance returned by a get() or load() method is persistent.
        - Detached: 
            - When an object is previously persistent but not associated with any session, it’s in detached state. 
            - Detached instances may be made persistent by calling update(), saveOrUpdate(), lock() or replicate(). 
            - The state of a transient or detached instance may also be made persistent as a new persistent instance by calling merge().
- What is hibernate configuration file?
    - Hibernate configuration file contains database specific configurations and used to initialize SessionFactory. 
    - We provide database credentials or JNDI resource information in the hibernate configuration xml file. 
    - Some other important parts of hibernate configuration file is Dialect information, so that hibernate knows the database type and mapping file or class details.
- What is hibernate caching? Explain Hibernate first level cache?
    - Hibernate caches query data to make our application faster. 
    - Hibernate Cache can be very useful in gaining fast application performance if used correctly. 
    - The idea behind cache is to reduce the number of database queries, hence reducing the throughput time of the application.
    - Hibernate first level cache is associated with the Session object. 
        - Hibernate first level cache is enabled by default and there is no way to disable it. 
        - However hibernate provides methods through which we can delete selected objects from the cache or clear the cache completely.
    - Any object cached in a session will not be visible to other sessions and when the session is closed, all the cached objects will also be lost.
- What are the benefits of Named SQL Query?
    - Hibernate Named Query helps us in grouping queries at a central location rather than letting them scattered all over the code.
    - Hibernate Named Query syntax is checked when the hibernate session factory is created, thus making the application fail fast in case of any error in the named queries.
    - Hibernate Named Query is global, means once defined it can be used throughout the application.
    - However one of the major disadvantages of Named query is that it’s hard to debug, because we need to find out the location where it’s defined.
- What is use of Hibernate Session merge() call?
    - Hibernate merge can be used to update existing values, however this method create a copy from the passed entity object and return it. 
    - The returned object is part of persistent context and tracked for any changes, passed object is not tracked. 
- What is ORM?
    - ORM stands for Object/Relational mapping. 
    - It is the programmed and translucent perseverance of objects in a Java application into the tables of a relational database using the metadata that describes the mapping between the objects and the database. 
    - It works by transforming the data from one representation to another.
- What is hibernate caching? Explain Hibernate first level cache?
    - As the name suggests, hibernate caches query data to make our application faster. 
    - Hibernate Cache can be very useful in gaining fast application performance if used correctly. 
    - The idea behind cache is to reduce the number of database queries, hence reducing the throughput time of the application.
    - Hibernate first level cache is associated with the Session object. 
        - Hibernate first level cache is enabled by default and there is no way to disable it. 
        - However, hibernate provides methods through which we can delete selected objects from the cache or clear the cache completely.
    - Any object cached in a session will not be visible to other sessions and when the session is closed, all the cached objects will also be lost.
- What does an ORM solution comprises of?
    - It should have an API for performing basic CRUD (Create, Read, Update, Delete) operations on objects of persistent classes
    - Should have a language or an API for specifying queries that refer to the classes and the properties of classes
    - Ability for specifying mapping metadata
    - It should have a technique for ORM implementation to 
        - interact with transactional objects to perform dirty checking, 
        - lazy association fetching, 
        - and other optimization functions
- Which design patterns are used in Hibernate framework?
    - Some of the design patterns used in Hibernate Framework are:
    - Domain Model Pattern –  An object model of the domain that incorporates both behavior and data.
    - Data Mapper – A layer of Mappers that moves data between objects and a database while keeping them independent of each other and the mapper itself.
    - Proxy Pattern for lazy loading
    - Factory pattern in SessionFactory
- What is Hibernate SessionFactory and how to configure it?
    - SessionFactory is the factory class used to get the Session objects. 
    - SessionFactory is responsible to read the hibernate Jenkins configuration parameters and connect to the database and provide Session objects. 
    - Usually an application has a single SessionFactory instance and threads servicing client requests obtain Session instances from this factory.
    - The internal state of a SessionFactory is immutable. 
    - Once it is created this internal state is set. 
    - This internal state includes all of the metadata about Object/Relational Mapping.
    - SessionFactory also provides methods to 
        - get the Class metadata and 
        - Statistics instance to get the stats of query executions, 
        - second level cache details etc.
- What are the advantages of Hibernate over JDBC?
    - Some of the important advantages of Hibernate framework over JDBC are:
        - Hibernate removes a lot of boiler-plate code that comes with JDBC API, 
            - the code looks more cleaner and readable.
        - Hibernate supports inheritance, associations and collections. 
            - These features are not present with JDBC API.
        - Hibernate implicitly provides transaction management, in fact most of the queries can’t be executed outside transaction. 
            - In JDBC API, we need to write code for transaction management using commit and rollback. 
        - JDBC API throws SQLException that is a checked exception, so we need to write a lot of try-catch block code. 
            - Most of the times it’s redundant in every JDBC call and used for transaction management. 
            - Hibernate wraps JDBC exceptions and throw JDBCException or HibernateException un-checked exception, so we don’t need to write code to handle it. 
            - Hibernate built-in transaction management removes the usage of try-catch blocks.
        - Hibernate Query Language (HQL) is more object oriented and close to java programming language. 
            - For JDBC, we need to write native sql queries.
        - Hibernate supports caching that is better for performance, JDBC queries are not cached hence performance is low.
        - Hibernate provide option through which we can create database tables too, for JDBC tables must exist in the database.
        - Hibernate configuration helps us in using JDBC like connection as well as JNDI DataSource for connection pool. 
            - This is very important feature in enterprise application and completely missing in JDBC API.
        - Hibernate supports JPA annotations, so code is independent of implementation and easily replaceable with other ORM tools. 
            - JDBC code is very tightly coupled with the application.
- What is Hibernate Validator Framework?
    - Data validation is integral part of any application. 
        - You will find data validation at presentation layer with the use of Javascript, then at the server side code before processing it. 
        - Also data validation occurs before persisting it, to make sure it follows the correct format.
    - Validation is a cross cutting task, so we should try to keep it apart from our business logic. 
        - That’s why JSR303 and JSR349 provide specification for validating a bean by using annotations. 
        - Hibernate Validator provides the reference implementation of both these bean validation specs.
- What is difference between Hibernate Session get () and load () method?
    - Hibernate session comes with different methods to load data from database. 
        - get and load are most used methods, at first look they seems similar but there are some differences between them.
            - get() loads the data as soon as it’s called whereas load() returns a proxy object and loads data only when it’s actually required, 
                - so load() is better because it support lazy loading.
        - Since load() throws exception when data is not found, we should use it only when we know data exists.
        - We should use get() when we want to make sure data exists in the database.
- What are the different levels of ORM quality? 
    - There are four levels defined for ORM quality.
        - Pure relational
        - Light object mapping
        - Medium object mapping
        - Full object mapping
- What is a pure relational ORM?
    - The entire application, including the user interface, is designed around the relational model and SQL-based relational operations.

Name some important interfaces of Hibernate framework?

Some of the important interfaces of Hibernate framework are:

SessionFactory (org.hibernate.SessionFactory): SessionFactory is an immutable thread-safe cache of compiled mappings for a single database. We need to initialize SessionFactory once and then we can cache and reuse it. SessionFactory instance is used to get the Session objects for database operations.

Session (org.hibernate.Session): Session is a single-threaded, short-lived object representing a conversation between the application and the persistent store. It wraps JDBC java.sql.Connection and works as a factory for org.hibernate.Transaction. We should open session only when it’s required and close it as soon as we are done using it. Session object is the interface between java application code and hibernate framework and provide methods for CRUD operations.

Transaction (org.hibernate.Transaction): Transaction is a single-threaded, short-lived object used by the application to specify atomic units of work. It abstracts the application from the underlying JDBC or JTA transaction. A org.hibernate.Session might span multiple org.hibernate.Transaction in some cases.

What is a meant by light object mapping?

The entities are represented as classes that are mapped manually to the relational tables.The code is hidden from the business logic using specific design patterns. This approach is successful for applications with a less number of entities, or applications with common, metadata-driven data models. This approach is most known to all.

What is a meant by medium object mapping?

The application is designed around an object model. The SQL code is generated at build time. And the associations between objects are supported by the persistence mechanism, and queries are specified using an object-oriented expression language. This is best suited for medium-sized applications with some complex transactions. Used when the mapping exceeds 25 different database products at a time.

What is meant by full object mapping?

Full object mapping supports sophisticated object modeling: composition, inheritance, polymorphism and persistence. The persistence layer implements transparent persistence; persistent classes do not inherit any special base class or have to implement a special interface. Efficient fetching strategies and caching strategies are implemented transparently to the application.

Benefits of ORM and Hibernate?

There are many benefits from these. Out of which the following are the most important one.

Productivity – Hibernate reduces the burden of developer by providing much of the functionality and let the developer to concentrate on business logic.

Maintainability – As hibernate provides most of the functionality, the LOC for the application will be reduced and it is easy to maintain. By automated object/relational persistence it even reduces the LOC.

Performance – Hand-coded persistence provided greater performance than automated one. But this is not true all the times. But in hibernate, it provides more optimization that works all the time there by increasing the performance. If it is automated persistence then it still increases the performance.

Vendor independence – Irrespective of the different types of databases that are there, hibernate provides a much easier way to develop a cross platform application.

How does hibernate code looks like?

Session session = getSessionFactory().openSession();

Transaction tx = session.beginTransaction();

MyPersistanceClass mpc = new MyPersistanceClass (“Sample App”);

session.save(mpc);

tx.commit();

session.close();

The Session and Transaction are the interfaces provided by hibernate. There are many other interfaces besides this.

What are different states of an entity bean?

An entity bean instance can exist is one of the three states.

Transient: When an object is never persisted or associated with any session, it’s in transient state. Transient instances may be made persistent by calling save (), persist() or saveOrUpdate(). Persistent instances may be made transient by calling delete ().

Persistent: When an object is associated with a unique session, it’s in persistent state. Any instance returned by a get() or load() method is persistent.

Detached: When an object is previously persistent but not associated with any session, it’s in detached state. Detached instances may be made persistent by calling update (), saveOrUpdate(), lock() or replicate(). The state of a transient or detached instance may also be made persistent as a new persistent instance by calling merge ().

What are the important benefits of using Hibernate Framework?

Some among the major benefits of using hibernate framework are as follows:

Provides support for JPA as well as XML annotations to make our code implementation independent.
Eliminates the boiler-plate code that comes with JDBC and manages resources focusing on business logic.
Provides a powerful query language which is similar to SQL. HQL however is a completely object-oriented and understands concepts like polymorphism, inheritance, and association.
Supports initialization using proxy objects and performs actual database queries only when they are necessary.
Hibernate cache helps us in getting better performance.
Cache helps us in acquiring better performance.
Hibernate is suitable for database vendor specific features as we can also execute native sql queries.
On the whole, Hibernate is the perfect choice for ORM tool in the current market as it contains all the features that are required for an ORM tool.
How to perform log hibernate that generates sql queries in log files?

To hibernate configuration to log SQL queries, we can set the following property

<property name="hibernate.show_sql">true</property>
- What do you think is Hibernate?
    - In Java applications, Hibernate is a commonly used object-relational mapping (ORM) tool. 
    - It is widely implemented in a variety of enterprise applications, especially in database operations. 
    - Hibernate enables users to create persistent classes based on principles like polymorphism, inheritance, composition, association, and collections.
- Are you aware of what derived properties are?
    - Derived properties are unmapped properties, but computed at runtime through evaluation of the expression. 
    - The expression is derived by using a formula corresponding to the element.
- Can you define the types of collection in Hibernate?
    - There are five types of collections -
        - Map
        - Array
        - List
        - Set
        - Bag 
- What is your understanding of Hibernate proxy?
    - The classes in Hibernate can be mapped as a proxy rather than a table. 
    - A proxy can be returned when the load in a session is called. 
    - Proxy comprises methods to load data. 
    - A proxy is often created for mapping classes to files.
- What according to you are the cache types in Hibernate?
    - Hibernate deploys two caches with respect to objects. 
        - First Level Cache : The first case is related to session object.
        - Second Level Cache : The second is related to SessionFactory. 
    - Hibernate, by default, uses the first on the basis of per-transaction. 
    - This cache is mainly used by Hibernate to lessen the SQL query numbers that need to be generated within a specific transaction.
- Can you explain Hibernate callback interfaces?
    - In Hibernate, callback interfaces are used to receive event notifications gathered from objects. 
    - An example for this is, when objects are loaded, as well as deleted, there is a generation of an event, the notification for which is sent through callback interfaces.
- What is your idea regarding session interface?
    - A session interface represents a hibernate session that performs manipulation of database entities. 
    - A session interface performs a variety of activities, which include - 
    - managing persistence states 2) fetching the persisted ones 3) management of transaction demarcations.
- What is Dirty Checking Feature in Hibernate?
    - Hibernate incorporates Dirty Checking Feature that permits developers and users to bypass time-consuming write actions. 
    - The Dirty Checking Feature changes or updates fields that need to be changed or updated, while keeping the remaining fields untouched and unchanged.
- How to create database applications in Java with the use of Hibernate?
    - Hibernate makes the creation of database applications in Java simple. 
    - The steps involved are 
        - we have to write the java object. 
        - A mapping file (XML) needs to be created that shows the relationship between class attributes and the database. 
        - Hibernate APIs have to be deployed in order to store the persistent objects.
- Can you share your views on mapping description files?
    - Mapping description files are used by Hibernate to configure functions. 
    - Mapping description files have the *.hbm extension, which facilitates the mapping between database tables and Java class. 
    - Whether to use mapping description files or not, entirely depends on business entities.
- What are your thoughts on file mapping in Hibernate?
    - File mapping is the core function of Hibernate. 
    - It is a prime tool in database mapping. 
    - Typically, the mapping takes place between attributes and classes. 
    - Application of tags, after mapping the files in a database, can indicate the primary key.
- Can you explain version field?
    - In case, offline information, backed by a database, is being changed, then data integrity at the application level is highly important. 
    - A versioning protocol, which is an advanced level of locking, is needed to support it. At this point in time, the use of version field comes in. However, the implementation and design process is up to users, primarily developers.
- What is different between Session and SessionFactory in Hibernate?
    - The main difference between Session and SessionFactory is that 
        - Session is a single-threaded, short-lived object 
        - Session provides first level cache        
        - SessionFactory is Immutable and shared by all Session.
        - SessionFactory also lives until the Hibernate is running. 
        - SessionFactory provides the Second level cache.
- What are your views on the function addClass?
    - The function addClass translates the name, basically from Java class to a file name. 
    - Subsequently, the Java ClassLoader loads the translated file as the input stream. 
    - The translation between one form to the other, precisely, from Java class to a file name, is called the addclass function.

- Can you explain the role of addDirectory() and addjar() methods?
    - The addDirectory() and addjar() methods in Hibernate allow users to load Hibernate documents. 
    - Both addDirectory() and addjar() methods play a significant role in simplifying a range of processes, such as layout, refactoring, configuration, and many more. 
        - AddDirectory() and addjar()  are greatly useful, especially when adding hibernate mapping with initialization files.
- What do you understand by Hibernate tuning?
    - Optimizing the performance of Hibernate applications is known as Hibernate tuning. 
    - Hibernate tuning is typically performed by employing data caching, session management, and SQL optimization.
- Can you explain what is Hibernate Query Language?
    - Hibernate deploys a high-performance query language known as Hibernate Query Language or HQL. 
    - HQL is almost similar to Structured Query Language (SQL) in many ways, except that it is entirely object-oriented. 
    - Rather than operating on columns and tables, Hibernate Query Language works with the properties of persistent objects.
- What is your understanding of Light Object Mapping?
    - The representation of entities are as classes, which are manually mapped to relational tables. 
    - Codes are masked from the domain logic applying particular design patterns. 
    - Light Object Mapping approach can be successful in case of applications where there are fewer entities, or for applications having data models that are metadata driven.
- What is N+1 SELECT problem in Hibernate?
    - The N+1 SELECT problem is a result of lazy loading and load on demand fetching strategy. 
    - In this case, Hibernate ends up executing N+1 SQL queries to populate a collection of N elements.
    - For example, if you have a List of N Items where each Item has a dependency on a collection of Bid object. 
    - Now if you want to find the highest bid for each item then Hibernate will fire 1 query to load all items and N subsequent queries to load Bid for each item.
    - So in order to find the highest bid for each item your application ends up firing N+1 queries.  
- What are some strategies to solve the N+1 SELECT problem in Hibernate?
    - Here are some strategies to solve the N+1 problem:
        - pre-fetching in batches, 
            - this will reduce the N+1 problem to N/K + 1 problem where K is the size of the batch
        - subselect fetching strategy
        - disabling lazy loading
##References 
- https://howtodoinjava.com/hibernate/how-hibernate-second-level-cache-works/
