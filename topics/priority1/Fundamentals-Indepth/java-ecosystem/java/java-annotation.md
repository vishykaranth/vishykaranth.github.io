---
layout: page
title: Class Loader
permalink: /java-annotation/
https://beginnersbook.com/2014/09/java-annotations/
---

##Java Annotation 

- Java Annotation is a tag that represents the metadata i.e. attached with class, interface, methods or fields to indicate some additional information which can be used by java compiler and JVM.
- Annotations were added to the java from JDK 5. Annotation has no direct effect on the operation of the code they annotate (i.e. it does not affect the execution of the program).

#### What’s the use of Annotations?
- Instructions to the compiler: There are three built-in annotations available in Java (@Deprecated, @Override & @SuppressWarnings) that can be used for giving certain instructions to the compiler. For example the @override annotation is used for instructing compiler that the annotated method is overriding the method. More about these built-in annotations with example is discussed in the next sections of this article.
- Compile-time instructors: Annotations can provide compile-time instructions to the compiler that can be further used by sofware build tools for generating code, XML files etc.
- Runtime instructions: We can define annotations to be available at runtime which we can access using java reflection and can be used to give instructions to the program at runtime. We will discuss this with the help of an example, later in this same post.

#### Annotations basics
- An annotation always starts with the symbol @ followed by the annotation name. The symbol @ indicates to the compiler that this is an annotation.
- For e.g. @Override
    - Here @ symbol represents that this is an annotation and the Override is the name of this annotation.
    
#### Where we can use annotations?
- Annotations can be applied to the classes, interfaces, methods and fields. For example the below annotation is being applied to the method.
~~~java
@Override
void myMethod() { 
    //Do something 
}
~~~
- What this annotation is exactly doing here is explained in the next section but to be brief it is instructing compiler that myMethod() is a overriding method which is overriding the method (myMethod()) of super class.

#### Built-in Annotations in Java
- Java has three built-in annotations:
    - @Override
        - While overriding a method in the child class, we should use this annotation to mark that method. This makes code readable and avoid maintenance issues, such as: while changing the method signature of parent class, you must change the signature in child classes (where this annotation is being used) otherwise compiler would throw compilation error. This is difficult to trace when you haven’t used this annotation.
            ~~~java
            public class MyParentClass {
            
                public void justaMethod() {
                    System.out.println("Parent class method");
                }
            }
            
            
            public class MyChildClass extends MyParentClass {
            
                @Override
                public void justaMethod() {
                    System.out.println("Child class method");
                }
            }
            ~~~   
         
    - @Deprecated
        - @Deprecated annotation indicates that the marked element (class, method or field) is deprecated and should no longer be used. The compiler generates a warning whenever a program uses a method, class, or field that has already been marked with the @Deprecated annotation. When an element is deprecated, it should also be documented using the Javadoc @deprecated tag, as shown in the following example. Make a note of case difference with @Deprecated and @deprecated. @deprecated is used for documentation purpose.
            ~~~java
            /**
             * @deprecated
             * reason for why it was deprecated
             */
            @Deprecated
            public void anyMethodHere(){
                // Do something
            }
            ~~~         
    - @SuppressWarnings
        - This annotation instructs compiler to ignore specific warnings. For example in the below code, I am calling a deprecated method (lets assume that the method deprecatedMethod() is marked with @Deprecated annotation) so the compiler should generate a warning, however I am using @@SuppressWarnings annotation that would suppress that deprecation warning.
            ~~~java
            @SuppressWarnings("deprecation")
                void myMethod() {
                    myObject.deprecatedMethod();
            }
            ~~~         
#### Creating Custom Annotations
- Annotations are created by using @interface, followed by annotation name as shown in the below example.
- An annotation can have elements as well. They look like methods. For example in the below code, we have four elements. We should not provide implementation for these elements.
- All annotations extends java.lang.annotation.Annotation interface. Annotations cannot include any extends clause.
    ~~~java
    
    import java.lang.annotation.Documented;
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Inherited;
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    import java.lang.annotation.Target;
     
    @Documented
    @Target(ElementType.METHOD)
    @Inherited
    @Retention(RetentionPolicy.RUNTIME)
    public @interface MyCustomAnnotation{
        int studentAge() default 18;
        String studentName();
        String stuAddress();
        String stuStream() default "CSE";
    }
    ~~~      