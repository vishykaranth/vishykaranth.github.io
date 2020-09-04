- Spring Boot is the next chapter of the Spring Framework. 
- Spring Boot is the Spring Framework
- Spring Boot is a new way to create Spring applications with ease.
- Spring Boot simplifies the way you develop, 
    - because it makes it easy to create production-ready
    - Spring-based applications that you can just run . 
    - You will find out that, with Spring Boot, you can create standalone applications that use an embedded server, making them 100% runnable applications. 
    - One of its best features is that Spring Boot is an “opinionated” technology in that it will help you follow the best practices for creating robust, extensible, and scalable Spring applications.
- Spring Boot has many features that make it suitable for:
    - Cloud Native Applications that follow the 12 factor patterns
    - Productivity increases by reducing time of development and deployment
    - Enterprise-production-ready Spring applications
    - Non-functional requirements, such as 
        - The Spring Boot Actuator
        - a module that brings metrics, 
        - health checks 
        - management easily 
        - Embedded containers for running web applications such as Tomcat, Undertow, Jetty, etc.)
- The term “microservices” is getting attention for creating scalable, highly available, and robust applications, and 
    - Spring Boot fits perfectly by allowing developers to focus only on the business logic and to leave the heavy lifting to the Spring Framework.
- Spring Boot Features
    - The SpringApplication class. 
        - In a Java Spring Boot application, the main method executes this singleton class.
        - This particular class provides a convenient way to initiate a Spring application.
    - Spring Boot allows you to create applications without requiring any XML configuration.
    - Spring Boot doesn’t generate code.
    - Spring Boot provides a fluent builder API through the SpringApplicationBuilder singleton class that allows you to create hierarchies with multiple application contexts. This particular feature is related to the Spring Framework and how it works internally. If you are a Spring developer already, you’ll learn more about this feature in the following chapters. If you are new to Spring and Spring Boot, you just need to know that you can extend Spring Boot to get more control over your applications.
    - Spring Boot offers you more ways to configure the Spring application events and listeners. This will be explained in more detail in the following chapters. I mentioned that Spring Boot is an “opinionated” technology, which means that Spring Boot will attempt to create the right type of application, either a web application (by embedding a Tomcat or Jetty container) or a single application.
    - The ApplicationArguments interface. Spring Boot allows you to access any application arguments. This is useful when you want to run your application with some parameters. For example, you can use --debug mylog.txt or --audit=true and have access to those values.
    - Spring Boot allows you to execute code after the application has started. The only thing you need to do is implement the CommandLineRunner interface and provide the implementation of the run(String ...args) method. A particular example is to initialize some records in a database as it starts or check on some services and see if they are running before your application starts.
    - Spring Boot allows you to externalize configurations by using an application.properties or application.yml file. 
    - You can add administration-related features, normally through JMX. 
        - You do this simply by enabling the spring.application.admin.enabled property in the application.properties or application.yml files.
    - Spring Boot allows you to have profiles that will help your application run in different environments.
    - Spring Boot allows you to configure and use logging very simply.
    - Spring Boot provides a simple way to configure and manage your dependencies by using starter poms. 
        - In other words, if you are going to create a web application, 
        - you only need to include the spring-boot-start-web dependency in your Maven pom or Gradle build file.
    - Spring Boot provides out-of-the-box non-functional requirements by using the Spring Boot Actuator, 
    - so you can see the health, memory, and so on, of your application.
    - Spring Boot provides @Enable<feature> annotations that help you to include, configure, and use technologies like 
    - databases (SQL and NoSQL), caching, scheduling, messaging, Spring integration, batching, and more.
- Gradle
    - You can use Gradle ( http://gradle.org/ ) to compile, test, and build Spring Boot apps. 
    - you need to have a minimum description for creating Spring Boot applications.    
    ~~~text
    buildscript {
        repositories {
            jcenter()
            maven { url "http://repo.spring.io/snapshot" }
            maven { url "http://repo.spring.io/milestone" }
        }
        dependencies {
            classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.1.RELEASE")
        }
    }
    ~~~  
- Using the Spring Initializr with UNIX cURL
    - curl -s https://start.spring.io/starter.zip -o myapp.zip
    - curl -s https://start.spring.io/starter.zip -o myapp.zip –d type=gradle-project
    - curl -s https://start.spring.io/starter.zip -o myapp.zip -d type=maven-project -d dependencies=web
    - curl -s https://start.spring.io/pom.xml -d packaging=war -o pom.xml -d dependencies=web,data-jpa
    - curl -s https://start.spring.io/build.gradle -o build.gradle -d dependencies=web,data-jpa
        - This command will generate only the build.gradle as a JAR 
            - (this is the default option, unless you use the -d packaging flag) and 
        - it will contain the same starters from the previous command. 
    - You can get more details about what other options you can set when executing the cURL command.
        - Just execute this command: $ curl start.spring.io .        