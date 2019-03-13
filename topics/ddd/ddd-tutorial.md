---
layout: page
title: Python Tutorial
permalink: /python-tutorial/
---

- [Domain Driven Design](#Domain-Driven-Design)


|  | ![](../../imgs/Domain%20Driven%20Design.png)  |  | 



## Domain Driven Design
- Focus on the core domain and domain logic.
- Base complex designs on models of the domain.
- Constantly collaborate with domain experts, in order to improve the application model and resolve any emerging domain-related issues.
- **Context**: The setting in which a word or statement appears that determines its meaning. Statements about a model can only be understood in a context.
- **Model**: A system of abstractions that describes selected aspects of a domain and can be used to solve problems related to that domain.
- **Ubiquitous** Language: A language structured around the domain model and used by all team members to connect all the activities of the team with the software.
- **Bounded Context**: A description of a boundary (typically a subsystem, or the work of a specific team) within which a particular model is defined and applicable.
### Domain 


### Building Blocks

- Entity: An object that is identified by its consistent thread of continuity, as opposed to traditional objects, which are defined by their attributes.
- Value Object: An immutable (unchangeable) object that has attributes, but no distinct identity.
- Domain Event: An object that is used to record a discrete event related to model activity within the system. While all events within the system could be tracked, a domain event is only created for event types which the domain experts care about.
- Aggregate: A cluster of entities and value objects with defined boundaries around the group. Rather than allowing every single entity or value object to perform all actions on its own, the collective aggregate of items is assigned a singular aggregate root item. Now, external objects no longer have direct access to every individual entity or value object within the aggregate, but instead only have access to the single aggregate root item, and use that to pass along instructions to the group as a whole. This practice correlates with many of the actual coding practices we’re covering in our design patterns series.
- Service: Essentially, a service is an operation or form of business logic that doesn’t naturally fit within the realm of objects. In other words, if some functionality must exist, but it cannot be related to an entity or value object, it’s probably a service.
- Repositories: Not be confused with common version control repositories, the DDD meaning of a repository is a service that uses a global interface to provide access to all entities and value objects that are within a particular aggregate collection. Methods should be defined to allow for creation, modification, and deletion of objects within the aggregate. However, by using this repository service to make data queries, the goal is to remove such data query capabilities from within the business logic of object models.
- Factories: As we’ve discussed through a number of design patterns articles already, DDD suggests the use of a factory, which encapsulates the logic of creating complex objects and aggregates, ensuring that the client has no knowledge of the inner-workings of object manipulation.


### Advantages of Domain-Driven Design
- **Eases Communication**: With an early emphasis on establishing a common and ubiquitous language related to the domain model of the project, teams will often find communication throughout the entire development life cycle to be much easier. Typically, DDD will require less technical jargon when discussing aspects of the application, since the ubiquitous language established early on will likely define simpler terms to refer to those more technical aspects.
- **Improves Flexibility**: Since DDD is so heavily based around the concepts of object-oriented analysis and design, nearly everything within the domain model will be based on an object and will, therefore, be quite modular and encapsulated. This allows for various components, or even the entire system as a whole, to be altered and improved on a regular, continuous basis.
- **Emphasizes Domain Over Interface**: Since DDD is the practice of building around the concepts of domain and what the domain experts within the project advise, DDD will often produce applications that are accurately suited for and representative of the domain at hand, as opposed to those applications which emphasize the UI/UX first and foremost. While an obvious balance is required, the focus on domain means that a DDD approach can produce a product that resonates well with the audience associated with that domain.

### Disadvantages of Domain-Driven Design
- **Requires Robust Domain Expertise**: Even with the most technically proficient minds working on development, it’s all for naught if there isn’t at least one domain expert on the team that knows the exact ins and outs of the subject area on which the application is intended to apply. In some cases, domain-driven design may require the integration of one or more outside team members who can act as domain experts throughout the development life cycle.
- **Encourages Iterative Practices**: While many would consider this an advantage, it cannot be denied that DDDpractices strongly rely on constant iteration and continuous integration in order to build a malleable project that can adjust itself when necessary. Some organizations may have trouble with these practices, particularly if their past experience is largely tied to less-flexible development models, such as the waterfall model or the like.
- **Ill-Suited for Highly Technical Projects**: While DDD is great for applications where there is a great deal of domain complexity (where business logic is rather complex and convoluted), DDD is not very well-suited for applications that have marginal domain complexity, but conversely have a great deal of technical complexity. Since DDD so heavily emphasizes the need for (and importance of) domain experts to generate the proper ubiquitous language and then domain model on which the project is based, a project that is incredibly technically complex may be challenging for domain experts to grasp, causing problems down the line, perhaps when technical requirements or limitations were not fully understood by all members of the team.

### Related Patterns 

#### Command Query Responsibility Segregation (CQRS)

- CQRS is an architectural pattern for separation of reads from writes, where the former is a Query and the latter is a Command. 
- Commands mutate state and are hence approximately equivalent to method invocation on aggregate roots/entities. 
- Queries query state but do not mutate it. 
- CQRS is a derivative architectural pattern from the design pattern called Command and Query Separation (CQS) which was coined by Bertrand Meyer. 
    - While CQRS does not require DDD, domain-driven design makes the distinction between commands and queries, explicit, around the concept of an aggregate root. 
    - The idea is that a given aggregate root has a method that corresponds to a command and a command handler invokes the method on the aggregate root. 
    - The aggregate root is responsible for performing the logic of the operation and yielding either a number of events or a failure (exception or execution result enumeration/number) response OR (if Event Sourcing (ES) is not used) just mutating its state for a persister implementation such as an ORM to write to a data store, while the command handler is responsible for pulling in infrastructure concerns related to the saving of the aggregate root's state or events and creating the needed contexts (e.g. transactions).
    
    
#### Event Sourcing (ES)
- An architectural pattern which warrants that your entities (as per Eric Evans’ definition) do not track their internal state by means of direct serialization or O/R mapping, but by means of reading and committing events to an event store. 
- Where ES is combined with CQRS and DDD, 
    - aggregate roots are responsible for thoroughly validating and applying commands (often by means having their instance methods invoked from a Command Handler), 
    - and then publishing a single or a set of events which is also the foundation upon which the aggregate roots base their logic for dealing with method invocations. 
- Hence, the input is a command and the output is one or many events which are transactionally (single commit) saved to an event store, and then often published on a message broker for the benefit of those interested (often the views are interested; they are then queried using Query-messages). When modeling your aggregate roots to output events, you can isolate the internal state event further than would be possible when projecting read-data from your entities, as is done in standard n-tier data-passing architectures. One significant benefit from this is that tooling such as axiomatic theorem provers (e.g. Microsoft Contracts or CHESS) are easier to apply, as the aggregate root comprehensively hides its internal state. Events are often persisted based on the version of the aggregate root instance, which yields a domain model that synchronizes in distributed systems around the concept of optimistic concurrency.


### Entity  
- Entities are concepts whose instances are uniquely identifiable
- Entities all have one immutable, read-only aspect or detail that acts as an identifier, 
    - but the thing with an entity is that we can change everything related to it (except its id) and it remains the same instance. 
    - It sounds a bit abstract and dry, but things are usually simple. 
    - It's quite easy to identify a concept that is an entity: the domain expert will refer to it as a single, identifiable item. 
    - Customer, Invoice, Order, Article, Post etc are examples of common concepts which are entities. 
    - As long as the business care about tracking each instance of that concept, we're dealing with an Entity.  
- Entity designates a 'bigger' concept, 
    - therefor we're dealing with an aggregate and we need to 'dig' deeper in order to find out its composition and its business rules. 
    - Another thing is an entity can't be part of an aggregate of another entity. 
    - But we can have links that is, an aggregate can 'point' to another entity. 
    - Maybe a better way of saying it is that when we have a domain relationship between 2 entities and one 'needs' the other one, 
    - the expression of it is a reference a.k.a the id of the other entity.
- An example is an Order is always placed by a Customer, 
    - so the Order entity has a relationship with the Customer concept, 
    - in the sense that we need to represent the Customer as being part of the Order (Create) aggregate. 
    - But since Customer is an Entity, only its id will be part of the Order aggregate.
    
### Value Objects 
- Value objects are simple or composite values that have a business meaning
- VOs usually are not treated with the same respect as the Entities, however they are equally important. 
    - It's not that hard to identify a VO, it's a concept that doesn't have an explicit identity detail. 
    - Actually, we can say that the identity of a VO is represented by all the values that VO has. 
    - Change something and it becomes another value.
- VO not only have values, but they incorporate business constraints that the values needs to respect. 
    - Basically, a VO is a 'smaller' concept, which is part of a (big concept) Entity. 
    - Most VO are small enough, 1-2 values but we can have bigger VO which combine other VOs.
- Let's take an example: A Price is a concept which is represented by 2 values: Amount and Currency (itself a VO). 
    - It doesn't have an explicit identity and most importantly, the business doesn't differentiate between one $5 and another $5.
- Another example is an Order's ordered product, a detail of the Order Entity. 
    - And no, I haven't seen any business talking about 'OrderLines', but they talk about an order's Items. 
    - The VO OrderItem is a composite of 3 things: Product, Quantity and Price. 
    - As a concept, Product is an entity and a VO can't contain Entities so what we'll have is a ProductId (or SKU) which itself is a VO. 
    - And it might seem that OrderItem has an identification detail, the product, used to identify a specific item from the order and making the item look like an entity, 
    - but the important thing is that OrderItem is a component of the Order, a minor concept which isn't used directly outside the Order. 
    - A business can't identify an OrderItem only by its 'id', it always needs the Order itself to act as a context for that 'id'. 
    - That's what makes it a VO, a true entity doesn't need another entity to be identifiable.
- Sometimes is tricky to identify the nature of a concept, it might look like an entity, 
    - however if it represents a minor concept which make sense only as part of another Entity, then we're dealing with a VO.        
    
#### Value Objects Properties
- Business Constraints - a VO communicates business semantics and usually this means those values need to respect business rules. A VO is always valid.
- Immutability - due to its nature, a VO never changes. 
    - It can't change or else it becomes another value. 
    - So, set its value(s) once and that's it.
- Usually a component of an aggregate - Business functionality deals with Entities, it doesn't care about a stand-alone VO instance. 
    - Ideally, all components of an aggregate are VOs.
- Semantics > Structure - Price has an amount and a currency. 
    - Discount has an amount and a currency. 
    - Are they the same concept? No, and the business cares about that. 
    - Even if 2 VOs have identical structure and business constraints they aren't the same and they should be implemented as different things. 
    - DRY doesn't apply here, never reuse VOs!
- Implementing a VO could be easy - an object with business rules enforced in the constructor plus getters. 
    - We can have methods but those methods should never change the VO, you can only create new VOs.
- VO implementation can be just a simple enum (think OrderStatus: Pending - InProgress - Completed).      

### Aggregates
- An aggregate by its definition is a group of 'lower level' concepts that need to always consistent. 
    - Technically speaking, an Agg is literally a group of value objects + Agg level domain rules. 
    - Note that these are rules that are required to maintain the aggregate consistency i.e rules that keep things valid and prevent the Agg entering an invalid state.
- From the domain rules point of view the Agg contains validity constraints, 
    - while from a consistency point of view the Agg defines a consistent area in our business model, which technically speaking is a Unit of Work. 
    - From another point of view, the Agg works only with VOs (these are the Agg's 'data') which themselves are a mix of data and business rules.
- So, which rules belong to the Agg? 
    - The ones which involve the role and the place of the VOs inside the Agg. 
    - Stuff like: "all Agg components are required". 
    - Never rules that involve the Agg itself as part of a bigger operation.
- Actually, the hardest and the most important part of identifying an Agg is to identify the relevant VOs, the Agg rules themselves are quite easy to spot. And if your business rule target the Agg as a whole then it's an 'outside of the Agg' rule that should be part of a Domain Service.

#### The Aggregate Root
- Looking again at the example Domain Objects - SalesOrderCheckout, PendingOrdersStream, WarehouseOrderDespatcher, OrderRefundProcessor there's still no obvious Aggregate Root; but that doesn't actually matter because the these Domain Objects have wildly separate responsibilities which don't seem to overlap.
- Functionally, there's no need for the SalesOrderCheckout to talk to the PendingOrdersStream because the job of the former is complete when it has added a new order to the Database; on the other hand, the PendingOrdersStream can retrieve new orders from the Database. These objects don't actually need to interact with each other directly (Perhaps an Event Mediator might provide notifications between the two, but I would expect any coupling between these objects to be very loose)
- Perhaps the Aggregate Root will be an IoC Container which injects one or more of those Domain Objects into a UI Controller, also supplying other infrastructure like EventMediator and Repository. Or perhaps it will be some kind of lightweight Orchestrator Service sitting on top of the Business Layer.
- The Aggregate root doesn't necessarily need to be a Domain Object. For the sake of keeping Separation of Concerns between Domain objects, it's generally a good thing when the aggregate root is a separate object with no business logic.

### Bounded Context
- an implementation of the separation of concerns principle
- A complex Domain can be considered a bunch of functionalities, but we like to group them into 'specialized' modules, where each module is responsible for a high level concern.
- If the Domain would be a company, the BCs would be its departments: accounting, HR, marketing, production etc. 
    - Each with its own responsibility, different but working together. 
    - And since DDD is about being Domain driven, we want to design our application similarly to how the real domain(business) is organized.
- each BC has boundaries, limits that isolate the model from other BCs(if there are more than one). 
    - the boundaries are the main benefit of a BC because they 'contain' a specific model, allowing us to focus on it. 
    - We can say that the main utility of BC is that it allows better decoupling, 
    - which is the most important trait of maintainability. 
    - Basically, we group together related models while keeping them separated from other models.
- And like everything else in DDD, we aren't the ones designing or deciding BCs, we identify them. 
    - The business already knows what responsibilities and limits each department has, we're just putting that information together.  
- The ubiquitous language is not just a DDD buzzword. 
    - It means that the business language has the same meaning in a certain context. 
    - Invoice means something to Accounting, but it can be just the name of a class for a programmer. 
    - Just because the name is identical doesn't mean it's the same concept. 
    - And Marketing can say Customer and Accounting can say Payer but they can both mean the same thing for the business as a whole.
- The important part here is that names are by themselves irrelevant. 
    - We need to understand the concept's definition that makes sense in that context in order to place a business case in a certain BC. 
    - Many times is quite easy, Create an order is part of ECommerce but sending the order's confirmation email is clearly part of a different BC, something like Marketing or Sales. 
    - How you send the email it's an implementation/infrastructure detail and we're not interested in that (at this point).
- Since the model of a BC makes sense only inside that BC we can't just use (refer) other BC's model directly. 
    - But in many cases we need information available in another BC. 
    - How can we get it, without breaking the boundaries?
- Enter Domain Events. 
    - They represent changes of the global business state, but they do have another, very useful technical trait: they are messages, simple data structures. 
    - They are no longer a model , they are simply information and they don't belong to a BC, they are domain wide valid.
- If one BC is interested in data from another BC, 
    - it will just subscribe to the relevant events and then it will use that data to maintain its very own read model (yes, that's CQRS in action). 
    - And since messages are data and not part of a model, boundaries are not violated. 
    - All this should be part of the BC's own infrastructure.
- But what about invoking a business case which is part of a different BC? 
    - When you need that, 
        - it means you're dealing with a business process, 
        - a sequence of business cases, however, 
        - even if a business case belongs to a BC, 
        - the process itself doesn't and we're going to have a process manager a.k.a saga in charge, 
        - which is a technical implementation detail. 
    - When we identify a process, we care about which business cases are part of it, the manager itself doesn't exist as part of the domain.
- A business case shouldn't invoke directly the next business case i.e it shouldn't act as a part-time project manager; 
    - in order to achieve decoupling and to maintain boundaries, 
    - business cases don't know anything about other cases, 
    - it's the process manager's job to know and to invoke the next case.
- The Bounded Context concept is very important for the maintainability of the app. 
    - It allows us to deal with relevant models that don't overlap. 
    - It's important to note that this is a logical grouping criteria and we can implement everything inside a monolith. 
    - But if the app needs to be scalable and we go distributed, then we may want to implement one BC per app/process/server.    
    
### Domain Services 
- Some business rules don't make sense to be part of an Aggregate. 
- For example, you want to check an account balance before debiting an account. 
- Or you want to see if a certain operation is allowed by the business rules (not to be mistaken for user authorization). 
- Or you want to perform a simple calculation according to domain rules. 
- Basically, any business rule required to move forward a business case, 
    - which doesn't belong to an aggregate should be a Domain Service. 
- This is the DDD term for business behaviour outside an aggregate or a value object.
- The easiest way is to simply check if the rules are having to do with business constraints required to maintain an aggregate invariants, including its value objects. If they're not, they're a DS, even if it seems related to that aggregate somehow. Many times you need to generate a value object that will be part of the aggregate. For ex: you need a Tax Calculator behaviour which will generate a Taxes value object which is a part of the Invoice Aggregate.    
- In other cases, you might need to allow an operation. Remember the transfer money example? Wha if we have a rule that says that you can't debit the account id the amount is lower than the balance? In this scenario we're actually dealing with 2 Domain Services: one is used to calculate the AccountBalance Value Object (VO) the other encapsulates the "amount must be greater than the balance" rule.
- In a business case we can use multiple DS and this might look like the Anaemic Domain. But it's not, the reason being that Aggregates should be as small as possible, not because someone said so, but because they should contain only the relevant behaviour for that model. Not every related behaviour. If something is 'outside' an Aggregate, then it's probably is a Domain Service.
- A DS is just a name signaling business behaviour. It doesn't mean you have to implement it in one way or another. In many cases you can implement all the domain services from that Bounded Context as (static) functions in one class, while in other cases you might want one class per DS. As with everything, it depends.
- A DS should be visible and consumed inside that Bounded Context only! Yes, you can define interfaces that can be used by your application service, they're great for testing, however their concrete implementation should be in the same BC as the business cases where it's used.
- Let's say we need to calculate tax but the domain expert says they're using some website to do it (and luckily for you, it provides an API and a client library). We need that functionality, but it's implementation is not part of our Domain. This is what I call an External Service. The easiest way to use it is to define an abstraction (interface part of your BC) that will act as if it's a Domain Service, however its implementation will be part of the Infrastructure and it will act as a wrapper around the client library. This is how we keep things decoupled. If later the calculation will be performed in-house, just implement a new class inside that BC and reconfigure the DI Container. Simple stuff.
- What if we need a service which is part of our app but part of another BC? For a monolith, you can cut corners and use it directly (assuming you weighted in the consequences). For a distributed app, I'd suggest the External Service approach. In the end, for the BC it's something outside its boundaries, it doesn't matter where the actual functionality is implemented as long as it's not inside the boundaries. For the BC's point of view, it's an external service.
        
### Application Service              
- We have all those domain models, aggregates and domain services but they are just parts of business cases, floating around. 
    - We need something to put things together, to orchestrate the fulfillment of a specific business case. 
    - We need a manager of sorts and in DDD that role is performed by an Application Service.    
- The simplest way to look at it is as the 'hosting environment' of a business case. 
    - It takes care of invoking the correct domain models and services, it mediates between Domain and Infrastructure and it shields any Domain model from the outside. 
    - Basically, in a DDD app, only the Application Service interacts directly with the Domain model. 
    - The rest of the application only sees the Application Service (interesting choice of name).
- Business functionality is expressed as business cases and the AS acts as a shell, encapsulating one business case (it should anyway, in practice we can host a simple process i.e more than one business case, but that's more of a pragmatic compromise). 
    - This means the UI/API or some higher layer talks to the AS responsible for that requested functionality and the AS gets to work. 
    - At most, the calling layer receives some result which shouldn't be nor contain a domain model i.e entities or value objects.
- It's important to note that an AS doesn't have any business behaviour. 
    - It knows only what model to use but not how that model works. 
    - So, the AS knows which Aggregate to invoke, it knows about some Domain Services that might be required, but that's it.  
- Let's continue the money transfer example. 
    - But now, we have the business rule that says the transfer can be done only if the account balance has enough money. 
    - So we're introducing 2 Domain Services: Calculate account balance and Can account be debitted both implemented as functions of a IDomainServices abstraction. 
    - But we also need an AccountBalance Value Object.
- For obvious reasons, the code is heavily simplified. 
    - Also, this is just one of the possible implementations. 
    - It's a matter of style in the end, as long as the code is clear and respects the domain model you can go wild with the implementation.      
- This particular approach uses a message driven architecture as well as Event Sourcing. We see how the AS orchestrates the business case; the domain model isn't aware of the command object, the event store or the result mediator. One thing that might be confusing is the branching, it almost looks like the AS needs to know the domain rules of what happens when an account balance is too low, however it doesn't :) . That's one of the "tell me if I can continue with the business case" type of situations and the AS uses a Domain Service to get an answer for that. Again, it's a matter of knowing who is responsible for something, not how something needs to be done. The AS never micro-manages.
- If you want to keep things even simpler, you can call this service directly from the UI/API layer, and make Execute() return a CommandResult. We can implement things as complex or as simple we want, as always, it depends on many factors.
- The Application Service is the host of a business case and acts as a mediator between the app and the domain model. As long as you keep the domain behaviour inside the dedicated patterns and none leaks into the AS, you're good to go. Implementations vary and there's not a recipe of how a "good" AS looks like. Just remember its role and respect the domain model, the actual code is up to you.       