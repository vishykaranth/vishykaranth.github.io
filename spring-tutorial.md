---
layout: page
title: Spring Tutorial
permalink: /spring-tutorial/
---



Table of contents:

- [Spring](#spring)
  - [Basic funtionality](#basic-funtionality)
  - [Advantages of Spring Framework](#Advantages-of-Spring-Framework)
  - [Spring Core Container Module] (#Spring-Core-Container-Module)
      - [Lists](#python-lists)

<a name='python'></a>

## Spring

- Spring enables developers to develop enterprise-class applications using POJOs.
    - The benefit of using only POJOs is that you do not need an EJB container product such as an application server but you have the option of using only a robust servlet container such as Tomcat or some commercial product.
- Spring is organized in a modular fashion.
    - Even though the number of packages and classes are substantial, you have to worry only about the ones you need and ignore the rest.
- Spring does not reinvent the wheel, instead it truly makes use of some of the existing technologies like
    - several ORM frameworks,
    - logging frameworks,
    - JEE,
    - Quartz
    - JDK timers
- Testing an application written with Spring is simple because environment-dependent code is moved into this framework. Furthermore, by using JavaBeanstyle POJOs, it becomes easier to use dependency injection for injecting test data.
- Spring's web framework is a well-designed web MVC framework, which provides a great alternative to web frameworks such as Struts or other over-engineered or less popular web frameworks.
- Spring provides a convenient API to translate technology-specific exceptions (thrown by JDBC, Hibernate, or JDO, for example) into consistent, unchecked exceptions.
- Lightweight IoC containers tend to be lightweight, especially when compared to EJB containers, for example. This is beneficial for developing and deploying applications on computers with limited memory and CPU resources.
- Spring provides a consistent transaction management interface that can scale down to a local transaction (using a single database, for example) and scale up to global transactions (using JTA, for example).

```java
package spring;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class GreetingsController {

    @GetMapping("/greeting")
    public String sayHello(
            @RequestParam(value = "name", required = false,
                          defaultValue = "World") String name, Model model) {
        model.addAttribute("name", name);
        return "hello";
    }
}
```


### Basic funtionality

- Spring is a light weight framework made easy for development with POJO.
- Dependency injection and programming to interface provides loose coupling.
- Supports declarative programming through aspects.
- Boilerplate code is reduced via aspects and templates.

### Advantages of Spring Framework

-Spring enables the developers to develop enterprise applications using POJOs (Plain Old Java Object). 
    -The benefit of developing the applications using POJO is, that we do not need to have an enterprise container such as an application server but we have the option of using a robust servlet container.
-Spring provides an abstraction layer on existing technologies like servlets, jsps, jdbc, jndi, rmi, jms and Java mail etc., to simplify the develpment process.
-Spring comes with some of the existing technologies like ORM framework, logging framework, J2EE and JDK Timers etc, Hence we don’t need to integrate explicitly those technologies.
-Spring WEB framework has a well-designed  web MVC framework, which provides a great alternate to lagacy web framework.
-Spring can eliminate the creation of the singleton and factory classes.
-Spring provides a consistent transaction management interface that can scale down to a local transaction and scale up to global transactions (using JTA).
-Spring framework includes support for managing business objects and exposing their services to the presentation tier components, so that the web and desktop applications can access the same objects.
-Spring framework has taken the best practice that have been proven over the years in several applications and formalized as design patterns.
-Spring application can be used for the development of different kind of applications, like standalone applications, standalone GUI applications, Web applications and applets as well.
-Spring supports both xml and anotation configurations.
-Spring Framework allows to develop standalone, desktop, 2 tire – n-tire architecture and distributed applications.
-Spring gives built in middleware services like Connection pooling, Transaction management and etc.,
-Spring provides a light weight container which can be activated without using webserver or application server.


### Spring Core Container Module
- The core container module provides the fundamental functionalities of the spring framework. 
    - In this module primary component is the BeanFactory, its sub interface ApplicationContext and its subclasses.
- Their primary goal of spring container is:
    - Working as IoC container
    - Resource management service
    - Lifecycle service
    - Additional services AOP container provides (Logging, Security, TXs, Testing, Fail-over handling)
- The two container super interfaces are:
    - BeanFactory
    - ApplicationContext
- BeanFactory container is used to lazy loads bean instances. In this container also bean instances are singleton by default but created when getBean() method is called.
- BeanFactory interface is having one sub class called
    - XmlBeanFactory (This class loads spring config file using filename)
        - BeanFactory f=new XmlBeanFactory(new FileInputStream(“beans.xml”));
        - BeanFactory f=new XmlBeanFactory(new FileSytsemResource(“beans.xml”));
- ApplicationContext container is used to eager load bean instances. In this container also bean instances are singleton by default but created at the time of container startup. It is more useful in when spring is used in web and other j2ee applications.
- ApplicationContext interface is having 3 sub classes they are:
    - FileSystemXmlApplicationContext (This class loads spring file from file system)
    - ClassPathXmlApplicationContext (This class loads spring file from classpath)
    - XmlWebApplicationContext (This class loads spring file in spring web & web mvc applications)