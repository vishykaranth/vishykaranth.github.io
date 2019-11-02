## Design Patterns Used in Spring Framework

### Dependency injection or inversion of control (IOC):
- Dependency injection is a technique in software engineering where an object can supply the dependencies of another object. Such dependency which can be used by an object is known as Service and the injection is the passing of the dependency to an object that uses it. Inversion of control or IOC is a design principle in software engineering through which custom-written computer program portions can receive the control flows from a generic framework. The Spring framework has an IOC container which is responsible for the creation of the objects, wiring the objects together, configuring these objects and handling the entire life cycle of these objects from their creation until they are completely destroyed. The container has the Dependency Injection (DI) which is used to manage the components present in an application. Such objects are known as Spring Beans. The dependency injection or IOC container is the main principle which is used in the spring framework for the decoupling process.

### Factory Design Pattern:
- The Spring framework uses the factory design pattern for the creation of the objects of beans by using the following two approaches.
- Spring BeanFactory Container: It is the simplest container present in the spring framework which provides the basic support for DI (Dependency Injection). We use the following interface to work with this container.[org.springframework.beans.factory.BeanFactory].
- Spring ApplicationContext Container: It is another container present in spring container which adds extra enterprise-specific functionality. These functionalities include the capability to resolve textual messages from a properties file and publishing application events to the attentive event listeners. We use the following interface to work with this container. [org.springframework.context.ApplicationContext]. Below are the most commonly used ApplicationContext implementations. FileSystemXmlApplicationContext (need to provide the full path of the XML bean configuration file to the constructor). ClassPathXmlApplicationContext (need to set CLASSPATH of the bean configuration XML file in order to load the metadata of the beans from an XML file). WebXmlApplicationContext (the container loads the XML file within a web application which has metadata of all beans).
~~~java 

package com.eduonix.springframework.applicationcontext;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class App {
	public static void main(String[] args) {
		ApplicationContext context = new FileSystemXmlApplicationContext(
				"C:/work/IOC Containers/springframework.applicationcontext/src/main/resources/bean-factory-config.xml");

		HelloApplicationContext obj = (HelloApplicationContext) context.getBean("helloApplicationContext");
		obj.getMsg();
	}
}

package com.eduonix.springframework.applicationcontext;
 
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;
 
public class App {
	public static void main(String[] args) {
		ApplicationContext context = new FileSystemXmlApplicationContext(
				"C:/work/IOC Containers/springframework.applicationcontext/src/main/resources/bean-factory-config.xml");
 
		HelloApplicationContext obj = (HelloApplicationContext) context.getBean("helloApplicationContext");
		obj.getMsg();
	}
}
~~~

### Proxy Design Pattern:
- In the proxy design pattern, a class is used to represent the functionality of another class. It is an example of a structural pattern. Here, an object is created that has an original object to interface its functionality to the outer world. Proxy design pattern is widely used in AOP, and remoting.

### Singleton Design Pattern:
- Singleton design pattern ensures that there will exist only the single instance of the object in the memory that could provide services. In the spring framework, the Singleton is the default scope and the IOC container creates exactly one instance of the object per spring IOC container. Spring container will store this single instance in a cache of singleton beans, and all following requests and references for that named bean will get the cached object as return bean. It is recommended to use the singleton scope for stateless beans. We can set up the bean scope as Singleton or prototype (which creates a new bean object for every new request) in the configuration XML file as shown below.

~~~xml
<!-- A bean definition with singleton scope -->
<bean id = "..." class = "..." scope = "singleton/prototype">
   <!-- collaborators and configuration for this bean go here -->
</bean>
<!-- A bean definition with singleton scope -->
<bean id = "..." class = "..." scope = "singleton/prototype">
   <!-- collaborators and configuration for this bean go here -->
</bean>
~~~

### Model View Controller (MVC):
- It is a design pattern which comes into picture when we use the spring framework for web programming. Spring MVC is known to be a lightweight implementation as controllers are POJOs against traditional servlets which makes the testing of controllers very comprehensive. A controller returns a logical view name and the view selection with the help of a separate ViewResolver. Therefore, Spring MVC controllers can be used along with different view technologies such as JSP, etc.

### Front Controller Design Pattern:
- The front controller design pattern is a technique in software engineering to implement centralized request handling mechanism which is capable of handling all the requests through a single handler. Such handler can perform the authentication, authorization, and logging or request tracking (i.e., pass the requests to the corresponding handlers). Spring framework provides support for the DispatcherServlet that ensure to dispatch an incoming request to your controllers.

### View Helper:
- Spring framework provides a large number of custom JSP tags as well as velocity macros which assist in the separation of code from the presentation (i.e., views).

### Template method:
- Spring framework provides a number of templates to kick start work and complete that piece of work as the best programming practice such as opening and closing connection for JDBC or JMS, etc. E.g., JdbcTemplate, JmsTemplate, and JpaTemplate.