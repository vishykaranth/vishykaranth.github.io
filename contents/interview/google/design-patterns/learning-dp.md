# Learning Plan for Design Patterns and Principles of Good Design

<!-- cSpell: ignore phptherightway Banas markdownlint Iluwatar Smok Dominik Liebler Pavel Loparev Oloruntoba Vlissides sublicense Wathan Laracon Podcast Liskov architecting Rafal Swidzinski behavioral craigctp Creational Reddit juanorozcov Orozco Villalobos Multiton Balking Podcasts Haines Vimeo Graphviz Amir kamranahmedse -->
<!-- Gist: https://gist.github.com/Pen-y-Fan/c99a94102fee9fd132265578b20885a1 -->
<!-- allow TOC links "message": "MD033/no-inline-html: Inline HTML [Element: a]" -->
<!-- markdownlint-disable MD033 -->

These learning resources primarily focus on programming using Good Design Principles and Design Patterns

- There is an emphasis on learning using PHP, although most patterns are universal to every object orientated language.
- All these resources are free (at the time of writing)

<!-- vscode-markdown-toc -->

## Table of Contents

- [1. Videos](#Videos)
- [2. Podcasts](#Podcasts)
- [3. Books, manuals and websites](#BooksManualsAndWebsites)
  - [Design Patterns](#DesignPatterns)
  - [Principles of Good Design](#Principles)
- [4. Tooling](#Tooling)
- [5. Conclusion](#Conclusion)
  - [Tips for Learning](#Tips)
- [6. Other Learning Plans](#OtherLearningPlans)
- [7. License](#License)

<!-- vscode-markdown-toc-config
    numbering=true
    autoSave=true
    /vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

## 1. <a name='Video'></a>Video (YouTube)

- [x] [Object Oriented Design](https://www.youtube.com/playlist?list=PLGLfVvz_LVvS5P7khyR4xDp7T9lCk9PgE) - Derek Banas (2012)
  - Derek creates several OO Designs using UML diagrams, before creating the code in Java.
    - Question: How does TDD differ from up front design using UML diagrams? [SO Q&A (1)](https://softwareengineering.stackexchange.com/questions/305093/why-is-it-inappropriate-to-use-uml-diagrams-to-plan-how-your-code-will-be-organi) & [SO Q&A (2)](https://softwareengineering.stackexchange.com/questions/82623/can-you-use-uml-in-a-tdd-project)
    - Agile manifesto: 'It only says "Just enough" and "we value working code more than documentation".'
    - [How Does TDD Affect Design?](https://www.jamesshore.com/Blog/How-Does-TDD-Affect-Design.html) - JamesShore.com (2014) - also see links to [LetsCodeJavascript.com](http://www.letscodejavascript.com/v3/blog/2014/05/how_does_tdd_affect_design)
- [x] [SOLID Principles of Object Oriented and Agile Design](https://www.youtube.com/watch?v=TMuno5RZNeE?t=745) - Bob Martin (2015) (start 12:25) YouTube 1h 10m
- [x] [Software Design Patterns and Principles (quick overview)](https://www.youtube.com/watch?v=WV2Ed1QTst8) - TechLead (2018) YouTube 11m23s. Keep it simple. TechLead briefly explained the design patterns he uses.
- [x] [Design Patterns Video Tutorial](https://www.youtube.com/playlist?list=PLF206E906175C7E07) - Derek Banas (2014) (Total playtime 7h 4m) - Covers 24 design patterns!
  > In my Design Patterns Video Tutorial I will cover all of the most common design patterns. I'll also explain when to use them and other topics on OOP design principles
- [x] [Chasing "Perfect"](https://www.youtube.com/watch?v=5DVDewOReoY) - Adam Wathan - Laracon EU 2015, YouTube 45 min
  - Real time coding, clean code, using tests at every step of the process.
  - This is also listed in my TDD plan. It is well worth another watch, after learning more about Patterns the refactoring made much more sense.
- [x] [PHP Design Patterns - Elements of Reusable Object-Oriented Software](https://www.youtube.com/playlist?list=PLGJDCzBP5j3xGaW0AGlaVHK2TMEr2XkP9) - Easy Learn Tutorial (2014) - covers 8 design patterns.
  > Design patterns are key to good PHP programming, and a fundamental to anyone wanting to learn PHP and become a better programmer. Design patterns are solutions for common problems that people have discovered and documented, so you don't have to re-invent the wheel every time you run into one of these problems on your next software engineering project.
- [x] [Design Patterns](https://www.youtube.com/playlist?list=PLO6785UZapFX5bO4ZZSO7Unn2uOTNFqfQ) - Smok (Rafal Swidzinski) (2019) - Java (~1h 10m) - A brief explanation of the 23 design patterns from GoF book.
  > Software Design Patterns have been around for a while. It is finally time to get to know them. In this video I introduce the most important terms, and explain what Design Patterns are about. So if you want to learn a Design Patterns - this is a good place to start.
- [x] [SOLID Principles](https://www.youtube.com/playlist?list=PL4CE9F710017EA77A) - craigctp (2012) (~14 min) 5 video series.
  > DevExpress CTO Messages for the SOLID object oriented development Principles.

## 2. <a name='Podcasts'></a>Podcasts

- [x] [16: Kent Beck - Tiny Decisions and Emergent Design](http://www.fullstackradio.com/16) - FullStackRadio #16 Podcast
  > In this episode, Adam talks to Kent Beck about Smalltalk vs. Java, low level design vs. big picture architecture, planning for the future vs. emergent design, and applying the principles of Extreme Programming in 2015.
- [x] [22: Corey Haines - The 4 Rules of Simple Design](http://www.fullstackradio.com/22) - FullStackRadio #22 Podcast
  > ... Every episode, Adam Wathan is joined by a guest \[Corey Haines\] to talk about everything from product design and user experience to unit testing and system administration.

## 3. <a name='BooksManualsAndWebsites'></a>Books, manuals and websites (inc. BlogPosts)

### <a name='DesignPatterns'></a>Design Patterns

- [ ] [Book:Design Patterns](https://en.wikipedia.org/wiki/Book:Design_Patterns) - Wikipedia, links to Wikipedia articles on design patterns (at the time of writing, the book is not available for download):
  - Overview:
    - [Design pattern](https://en.wikipedia.org/wiki/Software_design_pattern)
  - [Creational Patterns](https://en.wikipedia.org/wiki/Creational_pattern):
    - Creational pattern, Abstract factory pattern, Builder pattern, Factory method pattern, Lazy initialization, Multiton pattern, Object pool pattern, Prototype pattern, Resource Acquisition Is Initialization, Singleton pattern
  - [Structural patterns](https://en.wikipedia.org/wiki/Structural_pattern):
    - Structural pattern, Adapter pattern, Bridge pattern, Composite pattern, Decorator pattern, Facade pattern, Front Controller pattern, Flyweight pattern, Proxy pattern
  - [Behavioral patterns](https://en.wikipedia.org/wiki/Behavioral_pattern):
    - Behavioral pattern, Chain-of-responsibility pattern, Command pattern, Interpreter pattern, Iterator pattern, Mediator pattern, Memento pattern, Null Object pattern, Observer pattern, Publish/subscribe, Design pattern Servant, Specification pattern, State pattern, Strategy pattern, Template method pattern, Visitor pattern
  - [Concurrency patterns](https://en.wikipedia.org/wiki/Concurrency_pattern):
    - Concurrency pattern, Active object, Balking pattern, Messaging pattern, Double-checked locking, Asynchronous method invocation, Guarded suspension, Lock, Monitor, Reactor pattern, Readers-writer lock, Scheduler pattern, Thread pool pattern, Thread-local storage
- [ ] [Design Patterns](https://sourcemaking.com/design_patterns) - SourceMaking.com, detailed articles on code examples are given in Java, C++, PHP and Python
  > In software engineering, a design pattern is a general repeatable solution to a commonly occurring problem in software design. A design pattern isn't a finished design that can be transformed directly into code. It is a description or template for how to solve a problem that can be used in many different situations.
  - [Creational patterns](https://sourcemaking.com/design_patterns/creational_patterns) - 6 patterns (Abstract Factory, Builder, Factory Method, Object Pool, Prototype and Singleton)
    > In software engineering, creational design patterns are design patterns that deal with object creation mechanisms, trying to create objects in a manner suitable to the situation. The basic form of object creation could result in design problems or added complexity to the design. Creational design patterns solve this problem by somehow controlling this object creation.
  - [Structural patterns](https://sourcemaking.com/design_patterns/structural_patterns) - 8 patterns (Bridge, Composite, Decorator, Facade, Flyweight, Private Class Data and Proxy)
    > In Software Engineering, Structural Design Patterns are Design Patterns that ease the design by identifying a simple way to realize relationships between entities.
  - [Behavioral patterns](https://sourcemaking.com/design_patterns/behavioral_patterns) - 11 patterns (Chain of responsibility, Command, Interpreter, Iterator, Mediator, Memento, Null Object, Observer, State, Strategy, Template method, Visitor)
    > In software engineering, behavioral design patterns are design patterns that identify common communication patterns between objects and realize these patterns. By doing so, these patterns increase flexibility in carrying out this communication.
- [x] [Design Patterns](https://phptherightway.com/pages/Design-Patterns.html) - PHPTheRightWay.com (Factory, Singleton, Strategy, Front Controller, Model-View-Controller)
  > There are numerous ways to structure the code and project for your web application, and you can put as much or as little thought as you like into architecting. But it is usually a good idea to follow common patterns because it will make your code easier to manage and easier for others to understand.
- [ ] [DesignPatternsPHP](https://designpatternsphp.readthedocs.io/en/latest/) - Dominik Liebler (c2015)
  > This is a collection of known design patterns and some sample code how to implement them in PHP. Every pattern has a small list of examples (most of them from Zend Framework, Symfony2 or Doctrine2 as I’m most familiar with this software).
  - The book has all the documentation and code on [Github](https://github.com/domnikl/DesignPatternsPHP/), including tests!
- [ ] [PHP design patterns](https://phpenthusiast.com/blog/design-patterns) PHPEnthusiast (c2015)
- [ ] [Design Patterns Book](http://wiki.c2.com/?DesignPatternsBook) Original book by GoF, with links to articles
- [ ] [Design patterns implemented in Java](https://java-design-patterns.com/) - Iluwatar ([Github](https://github.com/iluwatar/java-design-patterns))
- [ ] [Design Patterns](https://github.com/PavelLoparev/design-patterns) - PavelLoparev (Github) (2017)
  > Contains examples of design patterns that are implemented using PHP
- [x] [GoFPatterns](https://www.gofpatterns.com/)
  > In this course you will learn what the GOF (Gang of Four) patterns are and how they help solve common problems encountered in object-oriented design.
  - Design Patterns are a software engineering concept describing recurring solutions to common problems in software design.
- [ ] [The Catalogue of Design Patterns](https://refactoring.guru/design-patterns) - Refactoring Guru (Partner site to SourceMaking.com)
- [ ] [Design Patterns for Humans](https://github.com/kamranahmedse/design-patterns-for-humans) - kamranahmedse (Github) (2018)
  > Ultra-simplified explanation to design patterns!
- [ ] [Awesome Software Design Patterns](https://github.com/DovAmir/awesome-design-patterns) - DovAmir (Github) (2019)
  > A curated list of software and architecture related design patterns.
- [ ] [Design Patterns in PHP](https://www.script-tutorials.com/design-patterns-in-php/) Andrew (c2019) - 23 design patterns with short PHP scripts.
  > Patterns in PHP. Today we will discuss design patterns in web development, more precisely – in PHP.

### [<a name='Principles'></a>Principles of Good Design

- [x] [SOLID principles of object-oriented programming](https://en.wikipedia.org/wiki/SOLID) - wikipedia (2-3 minutes)
  > SOLID is a mnemonic acronym for five design principles intended to make software designs more understandable, flexible and maintainable.
  - Single responsibility principle
    - A class should only have a single responsibility, that is, only changes to one part of the software's specification should be able to affect the specification of the class.
  - Open–closed principle
    - "Software entities ... should be open for extension, but closed for modification."
  - Liskov substitution principle
    - "Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program." See also design by contract.
  - Interface segregation principle
    - "Many client-specific interfaces are better than one general-purpose interface."
  - Dependency inversion principle
    - One should "depend upon abstractions, \[not\] concretions."
- [x] [GRASP (object-oriented design)](<https://en.wikipedia.org/wiki/GRASP_(object-oriented_design)>) - wikipedia (6-8 minutes)
  > General Responsibility Assignment Software Patterns (or Principles), abbreviated GRASP, consist of guidelines for assigning responsibility to classes and objects in object-oriented design.
  > The different patterns and principles used in GRASP are controller, creator, indirection, information expert, high cohesion, low coupling, polymorphism, protected variations, and pure fabrication.
- [x] [KISS principle](https://en.wikipedia.org/wiki/KISS_principle) - wikipedia (5-6 minutes)
  > KISS, an acronym for "keep it simple, stupid"
- [x] [Don't repeat yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) - wikipedia (3-4 minutes)
  > DRY is a principle of software development aimed at reducing repetition of software patterns, replacing it with abstractions or using data normalization to avoid redundancy.
- [x] [Software design pattern](https://en.wikipedia.org/wiki/Software_design_pattern) - wikipedia (25-32 minutes)
  > In software engineering, a software design pattern is a general, reusable solution to a commonly occurring problem within a given context in software design. It is not a finished design that can be transformed directly into source or machine code. It is a description or template for how to solve a problem that can be used in many different situations. Design patterns are formalized best practices that the programmer can use to solve common problems when designing an application or system.
- [x] [S.O.L.I.D: The First 5 Principles of Object Oriented Design](https://scotch.io/bar-talk/s-o-l-i-d-the-first-five-principles-of-object-oriented-design) - Samuel Oloruntoba (2015). 12-15 minutes read.
- [x] [S.O.L.I.D design principles for everyone](https://www.reddit.com/r/learnprogramming/comments/cr3m01/solid_design_principles_for_everyone/) - Reddit u/juanorozcov (AKA Juan Orozco Villalobos) posted links to his article on BrainsToBytes:
  - [ ] [The Single Responsibility Principle](https://www.brainstobytes.com/the-single-responsibility-principle/)
  - [ ] [The Open/Closed Principle](https://www.brainstobytes.com/the-open-closed-principle/)
  - [ ] [The Liskov Substitution Principle](https://www.brainstobytes.com/the-liskov-substitution-principle/)
  - [ ] [The Interface Segregation Principle](https://www.brainstobytes.com/interface-segregation-principle/)
  - [ ] [Dependency Injection and the famous DIP](https://www.brainstobytes.com/dependency-injection/)

## 4. <a name='Tooling'></a>Tooling

- [ ] [UMLet 14.3 Free UML Tool for Fast UML Diagrams](https://www.umlet.com/) - Used by Derek Banas in OO Design video, also see on-line version
- [ ] [PlantUML support for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml) - Needs Java (possibly Graphviz too), see the documentation.

## 5. <a name='Conclusion'></a>Conclusion

### <a name='Tips'></a>Tips for Learning

> Start shallow and broad. Skim as many as you can at the beginning. When you know what they do go deeper, only if you need to. - Smok

My take on this is to learn the different patterns, where they would be used, but do not memorise how to program every one. Once code is refactored and a pattern identified, then use the examples from above to implement the exact programming pattern.

> I think the problem with patterns is that often people do know them but don't know when to apply which. - Dominik Liebler (DesignPatternsPHP)

My take is the patterns need to be known and what context they are best used for.

## 6. <a name='OtherLearningPlans'></a>Other Learning Plans

The previous plan to Design Patterns is [Learning Plan for Test Driven Development (TDD)](https://gist.github.com/Pen-y-Fan/5a56a255a4733a04e10c58176cbe5f57)
