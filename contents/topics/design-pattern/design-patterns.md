### Abstract Factory Design Pattern
#### Intent
- Provide an interface for creating families of related or dependent objects without specifying their concrete classes.
- A hierarchy that encapsulates: many possible "platforms", and the construction of a suite of "products".
- The new operator considered harmful.
#### Problem
- If an application is to be portable, it needs to encapsulate platform dependencies.
- These "platforms" might include: windowing system, operating system, database, etc.
- Too often, this encapsulation is not engineered in advance,
- and lots of #ifdef case statements with options for all currently supported platforms begin to procreate like rabbits throughout the code.
#### Discussion
- Provide a level of indirection that abstracts the creation of families of related or dependent objects without directly specifying their concrete classes.
- The "factory" object has the responsibility for providing creation services for the entire platform family.
- Clients never create platform objects directly, they ask the factory to do that for them.
- This mechanism makes exchanging product families easy because the specific class of the factory object appears only once in the application - where it is instantiated.
- The application can wholesale replace the entire family of products simply by instantiating a different concrete instance of the abstract factory.
- Because the service provided by the factory object is so pervasive, it is routinely implemented as a Singleton.
#### Check list
- Decide if "platform independence" and creation services are the current source of pain.
- Map out a matrix of "platforms" versus "products".
- Define a factory interface that consists of a factory method per product.
- Define a factory derived class for each platform that encapsulates all references to the new operator.
- The client should retire all references to new, and use the factory methods to create the product objects.
#### Rules of thumb
- Sometimes creational patterns are competitors:
    - there are cases when either Prototype or Abstract Factory could be used profitably.
    - At other times they are complementary:
    - Abstract Factory might store a set of Prototypes from which to clone and return product objects,
    - Builder can use one of the other patterns to implement which components get built.
    - Abstract Factory, Builder, and Prototype can use Singleton in their implementation.
- Abstract Factory, Builder, and Prototype define a factory object that's responsible for knowing and creating the class of product objects, and make it a parameter of the system.
    - Abstract Factory has the factory object producing objects of several classes.
    - Builder has the factory object building a complex product incrementally using a correspondingly complex protocol.
    - Prototype has the factory object (aka prototype) building a product by copying a prototype object.
- Abstract Factory classes are often implemented with Factory Methods, but they can also be implemented using Prototype.
- Abstract Factory can be used as an alternative to Facade to hide platform-specific classes.
- Builder focuses on constructing a complex object step by step.
    - Abstract Factory emphasizes a family of product objects (either simple or complex).
    - Builder returns the product as a final step, but as far as the Abstract Factory is concerned, the product gets returned immediately.
- Often, designs start out using Factory Method (less complicated, more customizable, subclasses proliferate)
    - and evolve toward Abstract Factory, Prototype, or Builder (more flexible, more complex) as the designer discovers where more flexibility is needed.
### Builder Design Pattern
#### Intent
- Separate the construction of a complex object from its representation so that the same construction process can create different representations.
- Parse a complex representation, create one of several targets.
#### Problem
- An application needs to create the elements of a complex aggregate.
- The specification for the aggregate exists on secondary storage and one of many representations needs to be built in primary storage.
#### Discussion
- Separate the algorithm for interpreting (i.e. reading and parsing)
    - a stored persistence mechanism (e.g. RTF files)
    - from the algorithm for building
    - and representing one of many target products (e.g. ASCII, TeX, text widget).
    - The focus/distinction is on creating complex aggregates.
- The "director" invokes "builder" services as it interprets the external format.
    - The "builder" creates part of the complex object each time it is called and maintains all intermediate state.
    - When the product is finished, the client retrieves the result from the "builder".
- Affords finer control over the construction process.
    - Unlike creational patterns that construct products in one shot,
    - the Builder pattern constructs the product step by step under the control of the "director".
#### Check list
- Decide if a common input and many possible representations (or outputs) is the problem at hand.
- Encapsulate the parsing of the common input in a Reader class.
- Design a standard protocol for creating all possible output representations.
    - Capture the steps of this protocol in a Builder interface.
- Define a Builder derived class for each target representation.
- The client creates a Reader object and a Builder object, and registers the latter with the former.
- The client asks the Reader to "construct".
- The client asks the Builder to return the result.
#### Rules of thumb
- Sometimes creational patterns are complementary:
    - Builder can use one of the other patterns to implement which components get built.
    - Abstract Factory, Builder, and Prototype can use Singleton in their implementations.
- Builder focuses on constructing a complex object step by step.
    - Abstract Factory emphasizes a family of product objects (either simple or complex).
    - Builder returns the product as a final step,
    - but as far as the Abstract Factory is concerned,
    - the product gets returned immediately.
- Builder often builds a Composite.
- Often, designs start out using Factory Method
    - (less complicated, more customizable, subclasses proliferate)
    - and evolve toward Abstract Factory, Prototype, or Builder (more flexible, more complex) as the designer discovers where more flexibility is needed.
### Factory Method Design Pattern
#### Intent
- Define an interface for creating an object,
- but let subclasses decide which class to instantiate.
Factory Method lets a class defer instantiation to subclasses.
- Defining a "virtual" constructor.
- The new operator considered harmful.
#### Problem
- A framework needs to standardize the architectural model for a range of applications, but allow for individual applications to define their own domain objects and provide for their instantiation.
#### Discussion
- Factory Method is to creating objects as Template Method is to implementing an algorithm.
    - A superclass specifies all standard and generic behavior (using pure virtual "placeholders" for creation steps),
    - and then delegates the creation details to subclasses that are supplied by the client.
- Factory Method makes a design more customizable and only a little more complicated.
    - Other design patterns require new classes, whereas Factory Method only requires a new operation.
- People often use Factory Method as the standard way to create objects;
    - but it isn't necessary if: the class that's instantiated never changes,
    - or instantiation takes place in an operation that subclasses can easily override (such as an initialization operation).
- Factory Method is similar to Abstract Factory but without the emphasis on families.
- Factory Methods are routinely specified by an architectural framework,
    - and then implemented by the user of the framework.
#### Check list
- If you have an inheritance hierarchy that exercises polymorphism, consider adding a polymorphic creation capability by defining a static factory method in the base class.
- Design the arguments to the factory method.
    - What qualities or characteristics are necessary and sufficient to identify the correct derived class to instantiate?
- Consider designing an internal "object pool" that will allow objects to be reused instead of created from scratch.
- Consider making all constructors private or protected.
#### Rules of thumb
- Abstract Factory classes are often implemented with Factory Methods, but they can be implemented using Prototype.
- Factory Methods are usually called within Template Methods.
- Factory Method: creation through inheritance.
    - Prototype: creation through delegation (composition).
- Often, designs start out using Factory Method
    - (less complicated, more customizable, subclasses proliferate)
    - and evolve toward Abstract Factory, Prototype, or Builder (more flexible, more complex) as the designer discovers where more flexibility is needed.
- Prototype doesn't require subclassing, but it does require an Initialize operation.
    - Factory Method requires subclassing, but doesn't require Initialize.
- The advantage of a Factory Method is that it can return the same instance multiple times,
    - or can return a subclass rather than an object of that exact type.
- Some Factory Method advocates recommend that as a matter of language design (or failing that, as a matter of style) absolutely all constructors should be private or protected.
    - It's no one else's business whether a class manufactures a new object or recycles an old one.
- The new operator considered harmful.
    - There is a difference between requesting an object and creating one.
    - The new operator always creates an object, and fails to encapsulate object creation.
    - A Factory Method enforces that encapsulation,
    - and allows an object to be requested without inextricable coupling to the act of creation.
### Prototype Design Pattern
#### Intent
- Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.
- Co-opt one instance of a class for use as a breeder of all future instances.
- The new operator considered harmful.
#### Problem
- Application "hard wires" the class of object to create in each "new" expression.
#### Discussion
- Declare an abstract base class that specifies a pure virtual "clone" method, and, maintains a dictionary of all "cloneable" concrete derived classes. Any class that needs a "polymorphic constructor" capability: derives itself from the abstract base class, registers its prototypical instance, and implements the clone() operation.
- The client then, instead of writing code that invokes the "new" operator on a hard-wired class name, calls a "clone" operation on the abstract base class, supplying a string or enumerated data type that designates the particular concrete derived class desired.
#### Check list
- Add a clone() method to the existing "product" hierarchy.
- Design a "registry" that maintains a cache of prototypical objects. The registry could be encapsulated in a new Factory class, or in the base class of the "product" hierarchy.
- Design a factory method that: may (or may not) accept arguments, finds the correct prototype object, calls clone() on that object, and returns the result.
- The client replaces all references to the new operator with calls to the factory method.
#### Rules of thumb
- Sometimes creational patterns are competitors:
    - there are cases when either Prototype or Abstract Factory could be used properly.
    - At other times they are complementary:
    - Abstract Factory might store a set of Prototypes from which to clone and return product objects.
    - Abstract Factory, Builder, and Prototype can use Singleton in their implementations.
    - Abstract Factory classes are often implemented with Factory Methods, but they can be implemented using Prototype.
- Factory Method: creation through inheritance.
- Prototype: creation through delegation.
- Often, designs start out using Factory Method (less complicated, more customizable, subclasses proliferate)
    - and evolve toward Abstract Factory, Prototype, or Builder (more flexible, more complex) as the designer discovers where more flexibility is needed.
- Prototype doesn't require subclassing, but it does require an "initialize" operation.
    - Factory Method requires subclassing, but doesn't require Initialize.
- Designs that make heavy use of the Composite and Decorator patterns often can benefit from Prototype as well.
- Prototype co-opts one instance of a class for use as a breeder of all future instances.
- Prototypes are useful when object initialization is expensive, and you anticipate few variations on the initialization parameters. In this context, Prototype can avoid expensive "creation from scratch", and support cheap cloning of a pre-initialized prototype.
- Prototype is unique among the other creational patterns in that it doesn't require a class – only an object.
    - Object-oriented languages like Self and Omega that do away with classes completely rely on prototypes for creating new objects.
### Singleton Design Pattern
#### Intent
- Ensure a class has only one instance, and provide a global point of access to it.
- Encapsulated "just-in-time initialization" or "initialization on first use".
#### Problem
- Application needs one, and only one, instance of an object. Additionally, lazy initialization and global access are necessary.
#### Discussion
- Make the class of the single instance object responsible for creation, initialization, access, and enforcement.
    - Declare the instance as a private static data member. Provide a public static member function that encapsulates all initialization code, and provides access to the instance.
- The client calls the accessor function (using the class name and scope resolution operator) whenever a reference to the single instance is required.
- Singleton should be considered only if all three of the following criteria are satisfied:
    - Ownership of the single instance cannot be reasonably assigned
    - Lazy initialization is desirable
    - Global access is not otherwise provided for
- If ownership of the single instance, when and how initialization occurs, and global access are not issues, Singleton is not sufficiently interesting.
- The Singleton pattern can be extended to support access to an application-specific number of instances.
- The "static member function accessor" approach will not support subclassing of the Singleton class. If subclassing is desired, refer to the discussion in the book.
- Deleting a Singleton class/instance is a non-trivial design problem. See "To Kill A Singleton" by John Vlissides for a discussion.
#### Check list
- Define a private static attribute in the "single instance" class.
- Define a public static accessor function in the class.
- Do "lazy initialization" (creation on first use) in the accessor function.
- Define all constructors to be protected or private.
- Clients may only use the accessor function to manipulate the Singleton.
#### Rules of thumb
- Abstract Factory, Builder, and Prototype can use Singleton in their implementation.
- Facade objects are often Singletons because only one Facade object is required.
- State objects are often Singletons.
- The advantage of Singleton over global variables is that you are absolutely sure of the number of instances when you use Singleton,
    - and, you can change your mind and manage any number of instances.
- The Singleton design pattern is one of the most inappropriately used patterns.
    - Singletons are intended to be used when a class must have exactly one instance, no more, no less.
    - Designers frequently use Singletons in a misguided attempt to replace global variables.
    - A Singleton is, for intents and purposes, a global variable.
    - The Singleton does not do away with the global; it merely renames it.
- When is Singleton unnecessary?
    - Short answer:
        - most of the time.
    - Long answer:
        - when it's simpler to pass an object resource as a reference to the objects that need it,
        - rather than letting objects access the resource globally.
        - The real problem with Singletons is that they give you such a good excuse not to think carefully about the appropriate visibility of an object.
        - Finding the right balance of exposure and protection for an object is critical for maintaining flexibility.
- Our group had a bad habit of using global data, so I did a study group on Singleton.
    - The next thing I know Singletons appeared everywhere and none of the problems related to global data went away.
    - The answer to the global data question is not, "Make it a Singleton."
    - The answer is, "Why in the hell are you using global data?"
    - Changing the name doesn't change the problem.
    - In fact, it may make it worse because it gives you the opportunity to say, "Well I'm not doing that, I'm doing this" – even though this and that are the same thing.