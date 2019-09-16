Java Concurrency
====

[docs.oracle.com](https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)

-----

[Processes and Threads](https://docs.oracle.com/javase/tutorial/essential/concurrency/procthread.html)
-----

#### Process 

 - Self contained execution environment
 - Own memory space

## [Threads](https://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html)

 - Interprocess execution environment
 - Process have at least one

#### Threading Strategies

 - Instantiate own `Thread`s on demand
 - Pass tasks to an `Executor` (As described later)
  
#### Defining and starting a thread

 - Two methods:
     - Best practice: define `Runnable` class object
     - Other option: Extend `Thread` and execute `Thread.run()`

#### Pausing Threads
`Thread.sleep()`

 - Suspends thread for defined time
 - Not guaranteed to be precise. It's as precise as the OS may be
 - Cannot assume it will not be interrupted

#### Interrupts

 - Usually given with `InterruptedException`
 - Thread should stop and do something else
 - Very common for stopping threads

#### Join

 - Waits for a `Thread` to complete
 - Can have timeout like `Thread.sleep()` throwing `InterruptedException` when timed out
  
#### [SimpleThreads](https://docs.oracle.com/javase/tutorial/essential/concurrency/simple.html) Example

```java
public class SimpleThreads {

    // Display a message, preceded by
    // the name of the current thread
    static void threadMessage(String message) {
        String threadName =
            Thread.currentThread().getName();
        System.out.format("%s: %s%n",
                          threadName,
                          message);
    }

    private static class MessageLoop
        implements Runnable {
        public void run() {
            String importantInfo[] = {
                "Mares eat oats",
                "Does eat oats",
                "Little lambs eat ivy",
                "A kid will eat ivy too"
            };
            try {
                for (int i = 0;
                     i < importantInfo.length;
                     i++) {
                    // Pause for 4 seconds
                    Thread.sleep(4000);
                    // Print a message
                    threadMessage(importantInfo[i]);
                }
            } catch (InterruptedException e) {
                threadMessage("I wasn't done!");
            }
        }
    }

    public static void main(String args[])
        throws InterruptedException {

        // Delay, in milliseconds before
        // we interrupt MessageLoop
        // thread (default one hour).
        long patience = 1000 * 60 * 60;

        // If command line argument
        // present, gives patience
        // in seconds.
        if (args.length > 0) {
            try {
                patience = Long.parseLong(args[0]) * 1000;
            } catch (NumberFormatException e) {
                System.err.println("Argument must be an integer.");
                System.exit(1);
            }
        }

        threadMessage("Starting MessageLoop thread");
        long startTime = System.currentTimeMillis();
        Thread t = new Thread(new MessageLoop());
        t.start();

        threadMessage("Waiting for MessageLoop thread to finish");
        // loop until MessageLoop
        // thread exits
        while (t.isAlive()) {
            threadMessage("Still waiting...");
            // Wait maximum of 1 second
            // for MessageLoop thread
            // to finish.
            t.join(1000);
            if (((System.currentTimeMillis() - startTime) > patience)
                  && t.isAlive()) {
                threadMessage("Tired of waiting!");
                t.interrupt();
                // Shouldn't be long now
                // -- wait indefinitely
                t.join();
            }
        }
        threadMessage("Finally!");
    }
}
```

## [Intrinsic Locks & Synchronization](https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html)

 - Locks done through internal Java API known as `monitor`s
 - Allows only one thread to hold the lock
 - 2 threads accessing at once causes one to block until other gives up the `monitor`
 - Automatic acquisition and release when using **synchronize**'d methods
 - **Static** methods provide class level locks

#### Synchronized Statements

 - Allows `Object` to be specified as the intrinsic lock
Example using `lock1` and `lock2` as locks:

```java
public class MsLunch {
    private long c1 = 0;
    private long c2 = 0;
    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void inc1() {
        synchronized(lock1) {
            c1++;
        }
    }

    public void inc2() {
        synchronized(lock2) {
            c2++;
        }
    }
}
```
 - Must be careful with using this method
 - Make sure interleaving access to the fields is alright
 
#### Atomic Access
 - References and primitives are atomic writes
   - Except `long` and `double`
 - Reads are atomic for all **volatile** variables
 - Atomic access is more efficient than **synchronize**'d code
 - It requires more care, however
 
#### Deadlocks
 - Two `Thread`s waiting for each other to do something
 - `Thread`s will never end and must be interrupted

#### Starvation
 - **synchronize**'d method which is slow
 - One `Thread` calls method many times
 - Another `Thread` must now wait long periods before using method

#### Livelock
 - Two `Thread`s constantly responding to eachother
 - Not technically blocked, simply too busy to respond

#### Guarded Blocks
 - Coordinated actions between `Thread`s
 - Works by polling a condition that must be true to continue
 - Uses `Object.wait()` until interrupted and condition check can try again

#### Example:
```java
public synchronized void guardedJoy(){
    while ( ! joy ){
        try {
            wait();
        } catch ( InterruptedException e ) {}
    }
    System.out.println("We have joy!");
}
```
 - Note the `InterruptedException` is not the end of the loop, it just works to start the
   `Thread` back up and check the conditional once more
 - Invoking `wait()` inside **synchronize**'d methods is an easy way to acquire
   the intrinsic lock
 - `Object.notifyAll()` is usually better than `Object.notify()`
 - `Object.notify()` does not specify which `Thread` is woken up
 - Java Collections Framework has guarded block objects

## High Level Concurrency Objects
#### Lock Objects - Locking idioms to simplify concurrent applications
#### Executors - API for thread management
#### Concurrent Collections - Collections with reduced need of **synchronize**
#### Atomic Variables - Minimize syncing + memory consistency issues
#### ThreadLocalRandom - Psudorandom numbers from multiple `Thread`s

#### Lock Objects and the `Lock` Interface
 - Very similar to implicit locks
 - Support wait/notify through condition objects
 - Advantages:
  - Can back out of lock acquisition attempt
  - `Lock.tryLock()` backs out if not available or times out
  - `Lock.lockInterruptibly()` backs out if another `Thread` interrupts prior
    to acquisition

#### Executors - Interfaces, Pools and Fork/Join
 - 3 Interfaces:
  - Executor - Launcher for new tasks
  - ExecutorService - Subinterface of `Executor` with lifecycle management
    features
  - ScheduledExecutorService - Scheduled subinterface of `ExecutorService`

#### Executor Interface
 - `Executor.execute(Runnable)`
 - The `java.util.concurrent` implementation uses worker threads and
      more advanced functionality such as thread pools

#### ExecutorService Interface
 - `ExecutorService.submit(Runnable || Callable)`
 - Returns `Future` object
  - Retrieves `Callable` return value
  - Manages status of `Callable` and `Runnable` tasks
 - Methods for submitting `Collection`s of `Callable` tasks
 - Manages shutdown of the `Executor`

#### ScheduledExecutorService Interface
 - Executes `Runnable` or `Callable` tasks after a delay
 - `ScheduledExecutorService.scheduleAtFixedRate()`
 - `ScheduledExecutorService.scheduleWithFixedDelay()`

#### Thread Pools - `java.util.concurrent.Executors`
 - Minimizes `Thread` objects to instantiate
 - Alloc/Dealloc of `Thread` memory space is expensive
 - Fixed Thread Pool:
  - Specified number of `Thread`s
  - Terminated `Thread`s are replaced with new ones
  - Submitted via internal queue for when there are more tasks than available
    worker `Thread`s
  - Graceful degredation
   - Can't spin up more `Thread`s
   - Keeps execution within system limits, even when many tasks

#### Fork & Join
 - Implementation of `ExecutorService`
 - Built for multicore/many processors
 - Designed for small units of work built recursively
 - Goal is to use all available hardware power
 - Uses worker threads in a thread pool
 - Idle worker threads steal tasks from other busy threads

##### Basic Use
 - ForkJoinTask
  - Usually RecursiveTask or RecursiveAction Impl
```
if ( work is small enough )
    do it all
else
    split the work
    invoke this on each piece and wait for results
```

> Written with [StackEdit](https://stackedit.io/).