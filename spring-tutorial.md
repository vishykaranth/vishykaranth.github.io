---
layout: page
title: Spring Framework Tutorial
permalink: /spring-tutorial/
---


- [Spring](#spring)
  - [Basic funtionality](#basic-funtionality)
  - [Advantages of Spring Framework](#advantages-of-spring-framework)
  - [Spring Core Container Module](#spring-core-container-module)
    - [Bean’s lifecycle in BeanFactory](#bean’s-lifecycle-in-beanFactory)
    - [Bean’s lifecycle in ApplicationContext](#bean’s-lifecycle-in-applicationContext)
    - [Singleton Vs Prototype Vs Request Vs Session](#Singleton-Vs-Prototype-Vs-Request-Vs-Session)

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

- Spring enables the developers to develop enterprise applications using POJOs (Plain Old Java Object). 
    - The benefit of developing the applications using POJO is, that we do not need to have an enterprise container such as an application server but we have the option of using a robust servlet container.
- Spring provides an abstraction layer on existing technologies like servlets, jsps, jdbc, jndi, rmi, jms and Java mail etc., to simplify the develpment process.
- Spring comes with some of the existing technologies like ORM framework, logging framework, J2EE and JDK Timers etc, Hence we don’t need to integrate explicitly those technologies.
- Spring WEB framework has a well-designed  web MVC framework, which provides a great alternate to lagacy web framework.
- Spring can eliminate the creation of the singleton and factory classes.
- Spring provides a consistent transaction management interface that can scale down to a local transaction and scale up to global transactions (using JTA).
- Spring framework includes support for managing business objects and exposing their services to the presentation tier components, so that the web and desktop applications can access the same objects.
- Spring framework has taken the best practice that have been proven over the years in several applications and formalized as design patterns.
- Spring application can be used for the development of different kind of applications, like standalone applications, standalone GUI applications, Web applications and applets as well.
- Spring supports both xml and anotation configurations.
- Spring Framework allows to develop standalone, desktop, 2 tire – n-tire architecture and distributed applications.
- Spring gives built in middleware services like Connection pooling, Transaction management and etc.,
- Spring provides a light weight container which can be activated without using webserver or application server.


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
    

### Bean’s lifecycle in BeanFactory
- The container finds bean configuration in and instantiates bean.
- Using dependency injection spring populates all the properties as specified in spring xml file
- Bean implements BeanAware interface, the factory calls setBeanName() passing bean’s ID
- If Bean implements BeanFactoryAware interface the factory calls setBeanFactory(), passing instance of itself
- If there are any BeanPostProcessors associated with the bean, their PostProcessBeforeInitialization() methods are called
- If an init-method is specified for the bean they will be called
- Finally it there are any BeanPostProcessors associated with the bean, their postProcessAfterInitialization() methods are called
    - At this time bean is ready to use in application.
    - Bean instance remains in spring container when container shutdowns destroy-method specified in spring xml file is called or DisposableBean interface destroy() method is called.

### Bean’s lifecycle in ApplicationContext
- The only difference between BeanFactory and ApplicationContext is if bean implements ApplicationContextAware interface the setApplicationContext() method is called.

### Singleton Vs Prototype Vs Request Vs Session
- Singleton instances are created usually at the time of container startup in case of ApplicationContext and bean instances are created when getBean() method is called in case of BeanFactory.
- Singleton instance is a single instance through spring container.
- Prototype instances are created as many times as we request getBean() method in our client program.
- Prototype instances are not created at the time of container startup because we want to call getBean() method to get the bean instance, they are created on-demand.
- Request scope bean instance must be used in when using Spring web application framework. Their scope is the same bean instance is returned throughout the conversation.
- Session scope instances are per session scope. Whether you request single call HTML call or multiple calls on web application if you request from the same browser their instances are same.


### Initialization and Destruction
- When beans are instantiated and injections takes place it is necessary to call init() method to further initialize some more variables such as if we obtain DataSource it is necessary to call getConnection () method to get Connection object.
- Such initializations takes place in init() method.
- destroy() methods are called during container shutdown.
    - In this method we will close all DB objects.

### Constructor and Setter Injection
- Favor on Constructor Injection
- When dependent objects are mandatory then constructor injection is suitable.
- When dependent objects are optional then setter injection is suitable.
- If dependent objects are passed through constructor less amount of code consumes
- If dependent objects are passed through setter injection more amount of code occupies in both bean and spring config file.
- If any dependent objects are passed through constructor injection we need not have to write setter method for the property hence the property becomes passed into bean becomes immutable / cannot be modified in future in the bean.

### Disfavor on Constructor Injection
- Constructor’s parameters list increases, looks very length. Declaring such constructors in the bean is cumbersome/awkward.
- If there are several ways to create object of a bean then many constructors we need to write. If different constructors are written with different signatures and with more number of parameters then defining constructors itself takes lot of time, using them in spring config file is also tough.
- If constructor takes two parameters of the same type it may be difficult to determine what each parameter is.

### Bean Wiring
- Byname – Attempts to find bean in the container whose name (or id) is the same as name of the property.
- byType – Attempts to find single bean in the container whose type is matched with the property. type looking for injection.
- constructor – Tries to match one or more beans in the container with the parameters of one of the constructors of the bean being matched.
- Autodetect – Attempts to auto-wire by constructor first and then using byType.

### Configuring external file in spring
- Since every standalone – J2EE application are in need of using DB connection object preferably retrieved from the DataSource / Connection Pool object. Because connection obtained from the connection pool are efficient.
- Hence configuring DataSource is essential in J2EE frameworks like Struts, JSF, Spring, JPA, Hibernate etc.
- But while configuring DataSource even if we pass driver information such as driver class name, url, username and password is non-recommendable.
- Hence passing driver properties from an external jdbc.properties file is the most preferred mean.