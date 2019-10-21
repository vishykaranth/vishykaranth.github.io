## Decorator Pattern 

### Fundamentals 
- A single core component, several optional wrappers and a "common interface".
- Create a "common interface" that makes all classes interchangeable.
- Create a second level base class (Decorator) to support the optional wrapper classes.
- The Core class and Decorator class inherit from the same "common interface".
- The Decorator class declares a composition relationship to the "common interface", and this data member is initialized in its constructor.
- The Decorator class delegates to the "common interface".
- Define a **Decorator derived class** for each optional wrapper.
- **Decorator derived classes** 
    - implement their wrapper functionality 
    - delegate to the Decorator base class
- The client configures the type and ordering of Core and Decorator objects.

### Comparison with other Patterns 
- Adapter provides a different interface to its subject. 
    - Proxy provides the same interface. 
    - Decorator provides an enhanced interface.
- Adapter changes an object's interface, 
    - Decorator enhances an object's responsibilities. 
    - Decorator is thus more transparent to the client. 
    - **Decorator supports recursive composition**, which isn't possible with pure Adapters.
- Composite and Decorator have similar structure diagrams, 
    -reflecting the fact that both rely on recursive composition to organize an open-ended number of objects.
- A Decorator can be viewed as a degenerate Composite with only one component. 
    - However, a Decorator adds additional responsibilities - it isn't intended for object aggregation.
- Decorator is designed to let you add responsibilities to objects without subclassing. 
    - Composite's focus is not on wrapper but on representation. 
    - These intents are distinct but complementary. 
- Composite could use Chain of Responsibility to let components access global properties through their parent. 
    - It could also use Decorator to override these properties on parts of the composition.
- Decorator and Proxy have different purposes but similar structures. 
    - Both describe how to provide a level of indirection to another object, 
    - Implementations keep a reference to the object to which they forward requests.
- Decorator lets you change the skin of an object. 
    - Strategy lets you change the guts.

### Examples
- ![](imgs/decorator_pattern.png)