---
layout: page
title: spring-boot-6-swagger
permalink: /spring-boot-6-swagger/
---
Swagger
=======

Swagger is a very good open source tool for documenting REST based APIs provided by microservices. It provides very easy to use interactive documentation.

By the use of swagger annotation on REST endpoint, api documentation can be autogenerated and exposed over the web interface. Internal and external team can use web interface, to see the list of APIs and their inputs & error codes. They can even invoke the endpoints directly from web interface to get the results.

Swagger UI is a very powerful tool for your microservices consumers to help them understand set of endpoints provided by a given microservice.

Integrate Swagger into your microservices
-----------
Integrating swagger into Spring Boot based application should be straight forward. You need to add swagger dependencies into `build.gradle`, provide swagger configuration and finally make some tweaks into WebMvcConfig to allow swagger-ui into your project.

**build.gradle - add swagger dependencies.**
```xml
dependencies {
    compile('org.springframework.cloud:spring-cloud-starter-config')
    // https://mvnrepository.com/artifact/io.springfox/springfox-swagger2
    compile group: 'io.springfox', name: 'springfox-swagger2', version: '2.8.0'
    compile group: 'io.springfox', name: 'springfox-swagger-ui', version: '2.8.0'
```

**Second step is to define swagger configuration:**
SwaggerConfig.java.
```java
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.*;
import springfox.documentation.builders.*;
import springfox.documentation.service.*;

@Configuration
@EnableSwagger2
@EnableAutoConfiguration
public class SwaggerConfig {
    @Bean
    public Docket productApi() {
        return new Docket(DocumentationType.SWAGGER_2)
        .groupName("Product Service")
        .apiInfo(apiInfo())
        .select()
        .apis(RequestHandlerSelectors.basePackage("hello"))
        .paths(PathSelectors.any())
        .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
        .title("Product Service with Swagger")
        .description("Spring REST Sample with Swagger")
        .termsOfServiceUrl("http://www-03.ibm.com/software/sla/sladb.nsf/sla/bm?Open")
        .contact(new Contact("Munish Chandel", "","munish.chandel@outlook.com"))
        .license("Apache License Version 2.0")
        .licenseUrl("https://github.com/IBM-Bluemix/news-aggregator/blob/master/LICENSE")
        .version("1.0")
        .build();
    }
}

```
**Lastly, add the below WebMvcConfig to enable swagger UI**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    private static final Logger logger = LoggerFactory.getLogger(WebMvcConfig.class);
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        super.addResourceHandlers(registry);
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        /*registry.addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/");*/
    }
}

```
Now swagger is configured for use in your application.



Swagger annotations
-----------
Then the resource class can have annotations such as:

- `@Api`: To mark a resource as a Swagger resource
- `@ApiOperation`: Describes an operation or typically an HTTP method against a specific path
- `@ApiResponse`: To describe the response of a method
- `@ApiParam`: Additional metadata for operational parameters of a method

Maven plugin
-----------
A Maven plugin can be used to generate the swagger.yaml file based on the metadata placed on the code:
```xml
<build>
    ...
    <plugin>
        <groupId>com.github.kongchen</groupId>
        <artifactId>swagger-maven-plugin</artifactId>
        <version>3.1.5</version>
        <configuration>
            <apiSources>
                <apiSource>
                    <springmvc>false</springmvc>
                    <locations>org.jee8ng.users.boundary</locations>
                    <schemes>http</schemes>
                    <host>localhost:8081</host>
                    <basePath>/${project.build.finalName}/resources
                    </basePath>
                    <info>
                        <title>Users API</title>
                        <version>v1</version>
                        <description>Users rest endpoints</description>
                    </info>
                    <outputFormats>yaml</outputFormats>
                    <swaggerDirectory>${basedir}/src/main/webapp
                    </swaggerDirectory>
                </apiSource>
            </apiSources>
        </configuration>
        <executions>
            <execution>
                <phase>compile</phase>
                <goals>
                    <goal>generate</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    ...
</build>
```

The swaggerDirectory is where the swagger.yaml file gets generated. This way, it's possible to use a combination of plugins and annotations to create the Swagger Spec format with the desired output, such as JSON, configured here. The plugin and API details can be explored further on the Swagger website and on the GitHub pages of the plugin.















