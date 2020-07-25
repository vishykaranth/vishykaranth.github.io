---
layout: page
title: Spring Boot 
permalink: /spring-boot/
---

## Chapter 1
- Spring Boot brings 4 important features:
    - Automatic configuration
        — Spring Boot can automatically provide configuration for application functionality common to many Spring applications.
    - Starter dependencies
        — You tell Spring Boot what kind of functionality you need, and it will ensure that the libraries needed are added to the build.
    - The command-line interface
        — This optional feature of Spring Boot lets you write complete applications with just application code, but no need for a traditional project build.
    - The Actuator
        — Gives you insight into what’s going on inside of a running Spring Boot application.
        
### AUTO-CONFIGURATION
- Spring code has either Java configuration or XML configuration (or both) that enables certain supporting features and functionality for the application. 
- For example, if you’ve ever written an application that accesses a relational database with JDBC, you’ve probably configured Spring’s JdbcTemplate as a bean in the Spring application context. 
- Code would look like below 
~~~java
@Bean
public JdbcTemplate jdbcTemplate(DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}
~~~ 
- This very simple bean declaration creates an instance of JdbcTemplate, injecting it with its one dependency, a DataSource. 
- That means that you’ll also need to configure a DataSource bean so that the dependency will be met. 
- To complete this configuration scenario, suppose that you were to configure an embedded H2 database as the DataSource bean:
~~~java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.H2)
        .addScripts('schema.sql', 'data.sql')
        .build();
}
~~~       
- This bean configuration method creates an embedded database, specifying two SQL scripts to execute on the embedded database. 
- The build() method returns a Data-Source that references the embedded database.
- Neither of these two bean configuration methods is terribly complex or lengthy.
    - But they represent just a fraction of the configuration in a typical Spring application.
    - Moreover, there are countless Spring applications that will have these exact same methods. 
    - Any application that needs an embedded database and a JdbcTemplate will need those methods. 
    - In short, it’s boilerplate configuration.
- If it’s so common, then why should you have to write it?
- Spring Boot can automatically configure these common configuration scenarios. 
- If Spring Boot detects that you have the H2 database library in your application’s classpath, it will automatically configure an embedded H2 database. 
- If JdbcTemplate is in the classpath, then it will also configure a JdbcTemplate bean for you. 
- There’s no need for you to worry about configuring those beans. 
- They’ll be configured for you, ready to inject into any of the beans you write.
- There’s a lot more to Spring Boot auto-configuration than embedded databases and JdbcTemplate. 
- There are several dozen ways that Spring Boot can take the burden of configuration off your hands, 
    - including auto-configuration for the Java Persistence API (JPA), Thymeleaf templates, security, and Spring MVC. 