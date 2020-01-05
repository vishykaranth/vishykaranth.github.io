## Design Patterns

### Factory Method 
- Pros 
    - You avoid tight coupling between the creator and the concrete products.
    - Single Responsibility Principle. 
        - You can move the product creation code into one place in the program, making the code easier to support.
    - Open/Closed Principle. 
        - You can introduce new types of products into the program without breaking existing client code.
- Cons 
    - The code may become more complicated since you need to introduce a lot of new subclasses to implement the pattern. 
    - The best case scenario is when you’re introducing the pattern into an existing hierarchy of creator classes.
- Relations with Other Patterns
    - Many designs start by using Factory Method 
        - (less complicated and more customizable via subclasses) 
        - and evolve toward Abstract Factory, Prototype, or Builder 
        - (more flexible, but more complicated).
    - Abstract Factory classes are often based on a set of Factory Methods, 
        - but you can also use Prototype to compose the methods on these classes.
    - You can use Factory Method along with Iterator to 
        - let collection subclasses return different types of iterators that are compatible with the collections.
    - Prototype isn’t based on inheritance, so it doesn’t have its drawbacks. 
        - On the other hand, Prototype requires a complicated initialization of the cloned object. 
        - Factory Method is based on inheritance but doesn’t require an initialization step.
    - Factory Method is a specialization of Template Method. 
        - At the same time, a Factory Method may serve as a step in a large Template Method.    

### Abstract Factory
- Abstract Factory is a creational design pattern that lets you produce families of related objects without specifying their concrete classes.
- Pros
    - You can be sure that the products you’re getting from a factory are compatible with each other.
    - You avoid tight coupling between concrete products and client code.
    - Single Responsibility Principle. You can extract the product creation code into one place, making the code easier to support.
    - Open/Closed Principle. You can introduce new variants of products without breaking existing client code.
- Cons
    - The code may become more complicated than it should be, since a lot of new interfaces and classes are introduced along with the pattern.
    
- Relations with Other Patterns
    - Many designs start by using Factory Method (less complicated and more customizable via subclasses) and evolve toward Abstract Factory, Prototype, or Builder (more flexible, but more complicated).
    - Builder focuses on constructing complex objects step by step. Abstract Factory specializes in creating families of related objects. Abstract Factory returns the product immediately, whereas Builder lets you run some additional construction steps before fetching the product.
    - Abstract Factory classes are often based on a set of Factory Methods, but you can also use Prototype to compose the methods on these classes.
    - Abstract Factory can serve as an alternative to Facade when you only want to hide the way the subsystem objects are created from the client code.
    - You can use Abstract Factory along with Bridge. This pairing is useful when some abstractions defined by Bridge can only work with specific implementations. In this case, Abstract Factory can encapsulate these relations and hide the complexity from the client code.
    - Abstract Factories, Builders and Prototypes can all be implemented as Singletons.     
    
### Builder 
- Builder is a creational design pattern that lets you construct complex objects step by step. The pattern allows you to produce different types and representations of an object using the same construction code.
- Pros 
    - You can construct objects step-by-step, defer construction steps or run steps recursively.
    - You can reuse the same construction code when building various representations of products.
    - Single Responsibility Principle. You can isolate complex construction code from the business logic of the product.
- Cons 
    - The overall complexity of the code increases since the pattern requires creating multiple new classes.
- Relations with Other Patterns
    - Many designs start by using Factory Method (less complicated and more customizable via subclasses) and evolve toward Abstract Factory, Prototype, or Builder (more flexible, but more complicated).
    - Builder focuses on constructing complex objects step by step. Abstract Factory specializes in creating families of related objects. Abstract Factory returns the product immediately, whereas Builder lets you run some additional construction steps before fetching the product.
    - You can use Builder when creating complex Composite trees because you can program its construction steps to work recursively.
    - You can combine Builder with Bridge: the director class plays the role of the abstraction, while different builders act as implementations.
    - Abstract Factories, Builders and Prototypes can all be implemented as Singletons.

### Prototype
- Prototype is a creational design pattern that lets you copy existing objects without making your code dependent on their classes.
    - The Prototype pattern delegates the cloning process to the actual objects that are being cloned. The pattern declares a common interface for all objects that support cloning. This interface lets you clone an object without coupling your code to the class of that object. Usually, such an interface contains just a single clone method.
    - The implementation of the clone method is very similar in all classes. The method creates an object of the current class and carries over all of the field values of the old object into the new one. You can even copy private fields because most programming languages let objects access private fields of other objects that belong to the same class.
    - An object that supports cloning is called a prototype. When your objects have dozens of fields and hundreds of possible configurations, cloning them might serve as an alternative to subclassing.
- Pros
    - You can clone objects without coupling to their concrete classes.
    - You can get rid of repeated initialization code in favor of cloning pre-built prototypes.
    - You can produce complex objects more conveniently.
    - You get an alternative to inheritance when dealing with configuration presets for complex objects.
- Cons
    - Cloning complex objects that have circular references might be very tricky.
- Relations with Other Patterns
    - Many designs start by using Factory Method (less complicated and more customizable via subclasses) and evolve toward Abstract Factory, Prototype, or Builder (more flexible, but more complicated).
    - Abstract Factory classes are often based on a set of Factory Methods, but you can also use Prototype to compose the methods on these classes.
    - Prototype can help when you need to save copies of Commands into history.
    - Designs that make heavy use of Composite and Decorator can often benefit from using Prototype. Applying the pattern lets you clone complex structures instead of re-constructing them from scratch.
    - Prototype isn’t based on inheritance, so it doesn’t have its drawbacks. On the other hand, Prototype requires a complicated initialization of the cloned object. Factory Method is based on inheritance but doesn’t require an initialization step.
    - Sometimes Prototype can be a simpler alternative to Memento. This works if the object, the state of which you want to store in the history, is fairly straightforward and doesn’t have links to external resources, or the links are easy to re-establish.
    - Abstract Factories, Builders and Prototypes can all be implemented as Singletons.
    
### Singleton
- Singleton is a creational design pattern that lets you ensure that a class has only one instance, while providing a global access point to this instance.
- Pros
    - You can be sure that a class has only a single instance.
    - You gain a global access point to that instance.
    - The singleton object is initialized only when it’s requested for the first time.                   
- Cons
    - Violates the Single Responsibility Principle. The pattern solves two problems at the time.
    - The Singleton pattern can mask bad design, for instance, when the components of the program know too much about each other.
    - The pattern requires special treatment in a multithreaded environment so that multiple threads won’t create a singleton object several times.
    - It may be difficult to unit test the client code of the Singleton because many test frameworks rely on inheritance when producing mock objects. Since the constructor of the singleton class is private and overriding static methods is impossible in most languages, you will need to think of a creative way to mock the singleton. Or just don’t write the tests. Or don’t use the Singleton pattern.
- Relations with Other Patterns
    - A Facade class can often be transformed into a Singleton since a single facade object is sufficient in most cases.
    - Flyweight would resemble Singleton if you somehow managed to reduce all shared states of the objects to just one flyweight object. But there are two fundamental differences between these patterns:
    - There should be only one Singleton instance, whereas a Flyweight class can have multiple instances with different intrinsic states.
    - The Singleton object can be mutable. Flyweight objects are immutable.
    - Abstract Factories, Builders and Prototypes can all be implemented as Singletons.