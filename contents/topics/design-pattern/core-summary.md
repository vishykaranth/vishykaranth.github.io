### Structural Patterns 

#### Adapter Pattern 
- Adapter makes things work after they're designed; Bridge makes them work before they are.
- Bridge is designed up-front to let the abstraction and the implementation vary independently. 
    - Adapter is retrofitted to make unrelated classes work together.
- Adapter provides a different interface to its subject. 
    - Proxy provides the same interface. 
    - Decorator provides an enhanced interface.
- Adapter is meant to change the interface of an existing object. 
    -Decorator enhances another object without changing its interface. 
    - Decorator is thus more transparent to the application than an adapter is. 
    - As a consequence, Decorator supports recursive composition, which isn't possible with pure Adapters.
- Facade defines a new interface, whereas Adapter reuses an old interface. 
    - Remember that Adapter makes two existing interfaces work together as opposed to defining an entirely new one.
    
#### Bridge Pattern 
- Adapter makes things work after they're designed; Bridge makes them work before they are.
- Bridge is designed up-front to let the abstraction and the implementation vary independently. 
    - Adapter is retrofitted to make unrelated classes work together.
- State, Strategy, Bridge (and to some degree Adapter) have similar solution structures. 
    - They all share elements of the "handle/body" idiom. 
    - They differ in intent - that is, they solve different problems.
- The structure of State and Bridge are identical 
    - (except that Bridge admits hierarchies of envelope classes, whereas State allows only one). 
    - The two patterns use the same structure to solve different problems: 
    - State allows an object's behavior to change along with its state, 
    - while Bridge's intent is to decouple an abstraction from its implementation so that the two can vary independently.
- If interface classes delegate the creation of their implementation classes 
    - (instead of creating/coupling themselves directly), 
    - then the design usually uses the Abstract Factory pattern to create the implementation objects.    
    
#### Composite Pattern
##### Intent
- Compose objects into tree structures to represent whole-part hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.
- Recursive composition
- "Directories contain entries, each of which could be a directory."
- 1-to-many "has a" up the "is a" hierarchy
##### Facts  
- Composite and Decorator have similar structure diagrams, 
    - reflecting the fact that both rely on recursive composition to organize an open-ended number of objects.
- Composite can be traversed with Iterator. 
    - **Visitor** can apply an operation over a Composite. 
    - Composite could use **Chain of Responsibility** to let components access global properties through their parent. 
    - It could also use **Decorator** to override these properties on parts of the composition. 
    - It could use Observer to tie one object structure to another and State to let a component change its behavior as its state changes.
- Composite can let you compose a **Mediator** out of smaller pieces through recursive composition.
- Decorator is designed to let you add responsibilities to objects without subclassing. 
    - Composite's focus is not on embellishment but on representation. 
    - These intents are distinct but complementary. 
    - Consequently, Composite and Decorator are often used in concert.
- Flyweight is often combined with Composite to implement shared leaf nodes.

#### Decorator Design Pattern
##### Intent
- Attach additional responsibilities to an object dynamically. 
- Decorators provide a flexible alternative to subclassing for extending functionality.
- Client-specified embellishment of a core object by recursively wrapping it.
- Wrapping a gift, putting it in a box, and wrapping the box.

#### Check list
- Ensure the context is: a single core (or non-optional) component, several optional embellishments or wrappers, and an interface that is common to all.
- Create a "Lowest Common Denominator" interface that makes all classes interchangeable.
- Create a second level base class (Decorator) to support the optional wrapper classes.
- The Core class and Decorator class inherit from the LCD interface.
- The Decorator class declares a composition relationship to the LCD interface, and this data member is initialized in its constructor.
- The Decorator class delegates to the LCD object.
- Define a Decorator derived class for each optional embellishment.
- Decorator derived classes implement their wrapper functionality - and - delegate to the Decorator base class.
- The client configures the type and ordering of Core and Decorator objects.
#### Rules of thumb
- Adapter provides a different interface to its subject. 
    - Proxy provides the same interface. 
    - Decorator provides an enhanced interface.
- Adapter changes an object's interface, Decorator enhances an object's responsibilities. 
    - Decorator is thus more transparent to the client. 
    - As a consequence, Decorator supports recursive composition, which isn't possible with pure Adapters.
- Composite and Decorator have similar structure diagrams, reflecting the fact that both rely on recursive composition to organize an open-ended number of objects.
- A Decorator can be viewed as a degenerate Composite with only one component. 
    - However, a Decorator adds additional responsibilities - it isn't intended for object aggregation.
- Decorator is designed to let you add responsibilities to objects without subclassing. 
    - Composite's focus is not on embellishment but on representation. 
    - These intents are distinct but complementary. Consequently, Composite and Decorator are often used in concert.
- Composite could use Chain of Responsibility to let components access global properties through their parent. 
    - It could also use Decorator to override these properties on parts of the composition.
- Decorator and Proxy have different purposes but similar structures. 
    - Both describe how to provide a level of indirection to another object, and the implementations keep a reference to the object to which they forward requests.
- Decorator lets you change the skin of an object. 
    - Strategy lets you change the guts.
    
### Facade Design Pattern
#### Intent
Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.
Wrap a complicated subsystem with a simpler interface.
#### Problem
A segment of the client community needs a simplified interface to the overall functionality of a complex subsystem.

Discussion
Facade discusses encapsulating a complex subsystem within a single interface object. This reduces the learning curve necessary to successfully leverage the subsystem. It also promotes decoupling the subsystem from its potentially many clients. On the other hand, if the Facade is the only access point for the subsystem, it will limit the features and flexibility that "power users" may need.

The Facade object should be a fairly simple advocate or facilitator. It should not become an all-knowing oracle or "god" object.        