---
layout: page
title: threads
permalink: /threads/
---

## Core Classes 
- ExecutorService
- Executors
    - Creates a thread pool that reuses a fixed number of threads operating off a shared unbounded queue.  
    - At any point, at most nThreads threads will be active processing tasks.
    - If additional tasks are submitted when all threads are active, they will wait in the queue until a thread is available.
    - If any thread terminates due to a failure during execution prior to shutdown, a new one will take its place if needed to execute subsequent tasks.  
    - The threads in the pool will exist until it is explicitly shutdown.
- Runnable 
    - The Runnable interface should be implemented by any class whose instances are intended to be executed by a thread. 
    - The class must define a method of no arguments called run.
    - This interface is designed to provide a common protocol for objects that wish to execute code while they are active. 
    - For example, Runnable is implemented by class Thread.
    - Being active simply means that a thread has been started and has not yet been stopped.
    - In addition, Runnable provides the means for a class to be active while not subclassing Thread. 
    - A class that implements Runnable can run without subclassing Thread by instantiating a Thread instance and passing itself in as the target. 
    - In most cases, the Runnable interface should be used if you are only planning to override the run() method and no other Thread methods.
    - This is important because classes should not be subclassed unless the programmer intends on modifying or enhancing the fundamental behavior of the class.
- Callable 
    - This interface has the call() method. 
    - In this method, you have to implement the logic of a task. 
    - The Callable interface is a parameterized interface, meaning you have to indicate the type of data the call() method will return.
- Future 
    - This interface has some methods to obtain the result generated by a Callable object and to manage its state.
- Lock
- ThreadPoolExecutor 
    - provides a method that allows you to send to the executor a list of tasks and wait for the finalization of all the tasks in the list.
    - The Executor framework provides the ThreadPoolExecutor class to execute Callable and Runnable tasks with a pool of threads, 
        - which avoid you all the thread creation operations. 
        - When you send a task to the executor, it's executed as soon as possible, according to the configuration of the executor. 
        - There are used cases when you are not interested in executing a task as soon as possible. 
        - You may want to execute a task after a period of time or to execute a task periodically. 
        - For these purposes, the Executor framework provides the ScheduledThreadPoolExecutor class. 
            ~~~java
              ExecutorService executor = (ExecutorService)Executors.newCachedThreadPool();
              List<Task> taskList = new ArrayList<>();
              List<Future<Result>>resultList = executor.invokeAll(taskList);
              executor.shutdown();
              Result result = future.get();          
            ~~~
- Semaphores 
    - A semaphore is a counter that controls the access to one or more shared resources. 
    - This mechanism is one of the basic tools of concurrent programming and is provided by most of the programming languages.
- CountDownLatch  
    - Allows a thread to wait for the finalization of multiple operations.
- CyclicBarrier 
    - Allows the synchronization of multiple threads in a common point.
- Phaser 
    - Controls the execution of concurrent tasks divided in phases. 
    - All the threads must finish one phase before they can continue with the next one. 
- Exchanger 
    - Provides a point of data interchange between two threads.
            
