## How to Create Threads?
- Sub classing java.lang.Thread and Overriding its run method
    - The first way to customize a thread is to subclass java.lang.Thread (itself a java.lang.Runnable object) and 
    - always to override its empty run() method so that it  provides some meaningful concurrently-run services. 
    - Every instance of java.lang.Thread's subclass has its parallel executable code stored in the run() method. 
    - The run() method is actually defined in the java.lang.Runnable interface which the class java.lang.Thread implements it as an empty method.
- The  java.lang.Thread  class  provides a thread API and all the generic behavior for threads. 
    - The behaviors include starting, sleeping, running, yielding, and having a priority. 
- An instance of the subclass can then be allocated which is passive like all ordinary Java objects. 
    - It is switched to an active state by calling its start() method. 
    - The start() method spawns a new thread of control, and then returns (without waiting for the new thread to finish). 
    - Therefore, it is possible to have many different threads running at the same time, by calling each thread's start() method in turn. 
    - The start() method allocates system resources required for a thread, schedules the thread to run and invokes the run() method to start to be executed parallel. 
    - Inside the run()  method, other methods (of the same object as well of other objects) can be called without breaking up the parallelism. 
    - Once the run() method reaches its end, the object becomes passive again without the possibility to be restarted. 
    - But it can still provide non-parallel services by methods invoked in the usual way.  
    - For example: 
        ~~~java
            class CustomizedThread extends Thread {
              public CustomizedThread(String name){
                super(name);
              }
              public void run() {
                int next =0;
                while (next++!= 100) {
                  System.out.print(this.getName());
                  try {
                    Thread.sleep(100); 
                  } catch (InterruptedException	e){
                    System.out.println("Interrupted");
                    return;
                  }
                }
              }
            }
            
            public class Test {
              public static void main(String[] args) {
                Thread t1 = new CustomizedThread("1");
                Thread t2 = new CustomizedThread("2");
                t1.start();
                t2.start();
              }
            }
        ~~~

- The above call sleep causes the thread to wait for 100 milliseconds (1/10 of a second) after each iteration of the loop.
- The sleep method can throw an InterruptedException, and a thread's run method is not allowed to throw any exceptions (because Thread.run, which is being overridden, throws no exceptions). 
- Therefore, we must add code to catch that exception.
- A Runnable object, in which case a subsequent Thread.start invokes run of the supplied Runnable object. 
- If no Runnable is supplied, the default empty implementation of Thread.run() returns immediately.
- A String that serves as an identifier for the Thread. 
- This can be useful for tracing and debugging, but plays no other role. 
- If a thread is created without a name, one is automatically generated in the form Thread-n, where n is an integer
- The ThreadGroup in which the new Thread should be placed. If access to the ThreadGroup is not allowed, a SecurityException is thrown.

## User Thread and Daemo Thread
- Java makes a distinction between a user thread and another type of thread known as a daemon thread. Threads that work in the background to support the runtime environment are called daemon threads. The run() method for a daemon thread is typically an infinite loop that waits for a service request. For example, the clock handler thread, the idle thread, the garbage collector thread, the screen updater thread, and the garbage collector thread are all daemon threads.
- The difference between these two types of threads is straightforward: 
    - If the Java runtime determines that the only threads running in an application are daemon threads (i.e., there are no user threads in existence) the Java runtime promptly closes down the application, effectively stopping all daemon threads dead in their tracks. In order for an application to continue running, it must always have at least one live user thread. In all other respects the Java runtime treats daemon threads and user threads in exactly the same manner.

When the main() method initially receives control from the Java runtime it executes in the context of a user thread. It means that the main method thread is a user thread. All user threads are explicitly constructed and started from the main() method thread or by a thread originally started from main() method thread.  As long as the main() method thread (or any other user thread) remains alive the application will continue to execute. If you want all your threads to quit when main() completes you can set their status to daemon using setDaemon(true).

You can explicitly specify a thread to be a daemon thread by calling setDaemon(true) on a Thread object. Note that the setDaemon() method must be called before the thread's start() method is invoked.Once a thread has started executing (i.e., its start() method has been called) its daemon status cannot be changed. To determine if a thread is a daemon thread, use the accessor method isDaemon().

If a thread creates a new thread and does not call setDaemon() to explicitlly set the new thread's "daemon status", the new thread inherits the "daemon-status" of the thread that created it. All threads created by main method thread are user threads unless you explicitly sepcify a thread to be a daemo thread by calling setDaemon(true) on that Thread object.

Implementing the java.lang.Runnable Interface
A second way to implement threads - by implementing the java.lang.Runnable interface. The Runnable interface should be implemented by any class whose instances are intended to be executed by a thread. The class must define a method of no arguments called run .

This interface is designed to provide a common protocol for objects that wish to execute code while they are active. For example, Runnable is implemented by class Thread . Being active simply means that a thread has been started and has not yet been stopped.

In addition, Runnable provides the means for a class to be active while not subclassing Thread. A class that implements Runnable can run without subclassing Thread by instantiating a Thread instance and passing itself in as the target. Starting the thread causes the object's run method to be called in that separately executing thread.

In most cases, the Runnable interface should be used if you are only planning to override the run() method and no other Thread methods. This is important because classes should not be subclassed unless the programmer intends on modifying or enhancing the fundamental behavior of the class.

The following code is an example of implementing Runnable interface. 

public 
class SimpleRun implements Runnable{ public
  static void main(String argv[]){ SimpleRun
    r = new SimpleRun(); Thread
    t = new Thread(r); t.start();
    }
  public

  void run(){ for(int
    i= 0;i<100;i++)
      System.out.println(i);
  }
}
Deciding to Use the Runnable Interface
Class Thread itself implements Runnable. So, rather than supplying the code to be run in a Runnable and using it as an argument to a Thread constructor, you can create a subclass of Thread that overrides the run method to perform the desired actions.

However, the best default strategy is to define a Runnable as a separate class and supply it in a Thread constructor. Isolating code within a distinct class relieves you of worrying about any potential interactions of synchronized methods or blocks used in the Runnable with any that may be used by methods of class Thread. More generally, this separation allows independent control over the nature of the action and the context in which it is run: The same Runnable can be supplied to threads that are otherwise initialized in different ways, as well as to other lightweight executors. Also note that subclassing Thread precludes a class from subclassing any other class.

The way, "Sub classing java.lang.Thread and Overriding its run() method",   is easy to do but it means you cannot inherit from any other class, as Java only supports single inheritance. For example if you are creating a Button you cannot add threading via this method because a Button inherits from the AWT Button class and that uses your one shot at inheritance. Thus, if your class must subclass some other class (the most common example being Applet), you should use Runnable.