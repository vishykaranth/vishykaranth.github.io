---
layout: page
title: Threads Code Snippets 
permalink: /threads-code-snippets/
---

- Thread Management
    - Another concept related to concurrency is parallelism
    - Every Java program has at least one execution thread. 
        - When you run the program, the JVM runs this execution thread that calls the main() method of the program. 
    - When we call the start() method of a Thread object, we are creating another execution thread. 
        - Our program will have as many execution threads as calls to the start() method are made.
    - A Java program ends when all its threads finish (more specifically, when all its non-daemon threads finish). 
        - If the initial thread (the one that executes the main() method) ends, the rest of the threads will continue with their execution until they finish. 
        - If one of the threads use the System.exit() instruction to end the execution of the program, all the threads end their execution.    
        - Creating an object of the Thread class doesn't create a new execution thread. 
            - Also, calling the run() method of a class that implements the Runnable interface doesn't create a new execution thread. 
            - Only calling the start() method creates a new execution thread.  
        - The Thread class saves some information attributes that can help us to identify a thread, know its status, or control its priority. 
            - ID: This attribute stores a unique identifier for each Thread.
            - Name: This attribute store the name of Thread.
            - Priority: This attribute stores the priority of the Thread objects. 
                - Threads can have a priority between one and 10, where one is the lowest priority and 10 is the highest one. 
                - It's not recommended to change the priority of the threads, but it's a possibility that you can use if you want.
            - Status: This attribute stores the status of Thread. In Java, Thread can be in one of these six states: new, runnable, blocked, waiting, time waiting, or terminated.              