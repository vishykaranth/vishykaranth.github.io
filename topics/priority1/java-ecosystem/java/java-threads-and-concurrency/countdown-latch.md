## CountDownLatch
- CountDownLatch is a synchronization aid, introduced in Java 5. 
- Here the synchronization does not mean restricting access to a critical section. 
- But rather sequencing actions of different threads. 
- The type of synchronization achieved through CountDownLatch is similar to that of Join. 
- Assume that there is a thread "M" which needs to wait for other worker threads "T1", "T2", "T3" to complete its tasks Prior to Java 1.5, 
    - the way this can be done is, M running the following code
        ~~~java 
          T1.join();
          T2.join();
          T3.join();
        ~~~  
    - The above code makes sure that thread M resumes its work after T1, T2, T3 completes its work. 
    - T1, T2, T3 can complete their work in any order. 
    - The same can be achieved through CountDownLatch, where T1,T2, T3 and thread M share same CountDownLatch object.
    - "M" requests :  countDownLatch.await(); 
    - where as "T1","T2","T3" does  countDownLatch.countdown(); 
    - One disadvantage with the join method is that M has to know about T1, T2, T3. 
        - If there is a new worker thread T4 added later, then M has to be aware of it too. 
        - This can be avoided with CountDownLatch. 
        - After implementation the sequence of action would be [T1,T2,T3](the order of T1,T2,T3 could be anyway) -> [M]
- Using a CountDownLatch we can cause a thread to block until other threads have completed a given task.
- CountDownLatch in Java is a kind of synchronizer which allows one Thread  to wait for one or more Threads before starts processing. 
    - This is very crucial requirement and often needed in server side core Java application and having this functionality built-in as CountDownLatch greatly simplifies the development. 
    - CountDownLatch in Java is introduced on Java 5 along with other concurrent utilities like CyclicBarrier, Semaphore, ConcurrentHashMap and BlockingQueue in java.util.concurrent package. 
    - You can also implement same functionality using  wait and notify mechanism in Java but it requires lot of code and getting it write in first attempt is tricky,  With CountDownLatch it can  be done in just few lines. 
    - CountDownLatch also allows flexibility on number of thread for which main thread should wait, 
    - It can wait for one thread or n number of thread, there is not much change on code.  
    - Key point is that you need to figure out where to use CountDownLatch in Java application which is not difficult if you understand What is CountDownLatch in Java, 
    - What does CountDownLatch do and How CountDownLatch works in Java. 
- CountDownLatch has a counter field, which you can decrement as we require. 
    - We can then use it to block a calling thread until it’s been counted down to zero.
- If we were doing some parallel processing, we could instantiate the CountDownLatch with the same value for the counter as a number of threads we want to work across. 
- Then, we could just call countdown() after each thread finishes, guaranteeing that a dependent thread calling await() will block until the worker threads are finished.
    ~~~java
        public class CountDownLatchDemo implements Runnable{
                private List<String> logs;
                private CountDownLatch countDownLatch;
        
                public CountDownLatchDemo(List<String> logs, CountDownLatch countDownLatch) {
                    this.logs = logs;
                    this.countDownLatch = countDownLatch;
                }
        
                @Override
                public void run() {
                    doSomeWork();
                    logs.add("Thread completed !!");
                    countDownLatch.countDown();
                }
        
                private void doSomeWork() {
                    System.out.println("Some Work !!");
                }
        }
  

        @Test
        public void waitUntilCompletion() throws InterruptedException {
    
            List<String> logs = Collections.synchronizedList(new ArrayList<>());
            int THREAD_COUNT = 3;
            CountDownLatch countDownLatch = new CountDownLatch(THREAD_COUNT);
    
            //Create Threads
            List<Thread> threads = Stream
                    .generate(() -> new Thread(new CountDownLatchDemo(logs, countDownLatch)))
                    .limit(THREAD_COUNT)
                    .collect(toList());
    
            //Start Threads
            threads.forEach(Thread::start);
    
            //Wait until completion
            countDownLatch.await();
            logs.add("Job Completed !!");
    
            assertThat(logs).containsExactly("Thread completed !!", "Thread completed !!", "Thread completed !!", "Job Completed !!");
        }
    
    ~~~

- A Pool of Threads Waiting to Begin
    - If we took the previous example, 
    - but this time started thousands of threads instead of five, 
    - it’s likely that many of the earlier ones will have finished processing before we have even called start() on the later ones. 
    - This could make it difficult to try and reproduce a concurrency problem, 
    - as we wouldn’t be able to get all our threads to run in parallel.
    - To get around this, let’s get the CountdownLatch to work differently than in the previous example. 
    - Instead of blocking a parent thread until some child threads have finished, 
    - we can block each child thread until all the others have started.
    ~~~java 
        public class WaitingWorker implements Runnable {
         
            private List<String> outputScraper;
            private CountDownLatch readyThreadCounter;
            private CountDownLatch callingThreadBlocker;
            private CountDownLatch completedThreadCounter;
         
            public WaitingWorker(
              List<String> outputScraper,
              CountDownLatch readyThreadCounter,
              CountDownLatch callingThreadBlocker,
              CountDownLatch completedThreadCounter) {
         
                this.outputScraper = outputScraper;
                this.readyThreadCounter = readyThreadCounter;
                this.callingThreadBlocker = callingThreadBlocker;
                this.completedThreadCounter = completedThreadCounter;
            }
         
            @Override
            public void run() {
                readyThreadCounter.countDown();
                try {
                    callingThreadBlocker.await();
                    doSomeWork();
                    outputScraper.add("Counted down");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    completedThreadCounter.countDown();
                }
            }
        }
 
    @Test
    public void whenDoingLotsOfThreadsInParallel_thenStartThemAtTheSameTime()
     throws InterruptedException {
      
        List<String> outputScraper = Collections.synchronizedList(new ArrayList<>());
        CountDownLatch readyThreadCounter = new CountDownLatch(5);
        CountDownLatch callingThreadBlocker = new CountDownLatch(1);
        CountDownLatch completedThreadCounter = new CountDownLatch(5);
        List<Thread> workers = Stream
          .generate(() -> new Thread(new WaitingWorker(
            outputScraper, readyThreadCounter, callingThreadBlocker, completedThreadCounter)))
          .limit(5)
          .collect(toList());
     
        workers.forEach(Thread::start);
        readyThreadCounter.await(); 
        outputScraper.add("Workers ready");
        callingThreadBlocker.countDown(); 
        completedThreadCounter.await(); 
        outputScraper.add("Workers complete");
     
        assertThat(outputScraper)
          .containsExactly(
            "Workers ready",
            "Counted down",
            "Counted down",
            "Counted down",
            "Counted down",
            "Counted down",
            "Workers complete"
          );
    }
    ~~~
    
- This pattern is really useful for trying to reproduce concurrency bugs, as can be used to force thousands of threads to try and perform some logic in parallel.
- Terminating a CountdownLatch Early
    - Sometimes, we may run into a situation where the Workers terminate in error before counting down the CountDownLatch. 
    - This could result in it never reaching zero and await() never terminating:
    ~~~java 
        @Override
        public void run() {
            if (true) {
                throw new RuntimeException("Oh dear, I'm a BrokenWorker");
            }
            countDownLatch.countDown();
            outputScraper.add("Counted down");
        }
        
        @Test
        public void whenFailingToParallelProcess_thenMainThreadShouldGetNotGetStuck()
          throws InterruptedException {
          
            List<String> outputScraper = Collections.synchronizedList(new ArrayList<>());
            CountDownLatch countDownLatch = new CountDownLatch(5);
            List<Thread> workers = Stream
              .generate(() -> new Thread(new BrokenWorker(outputScraper, countDownLatch)))
              .limit(5)
              .collect(toList());
         
            workers.forEach(Thread::start);
            countDownLatch.await();
        }
    ~~~
- Clearly, this is not the behavior we want – it would be much better for the application to continue than infinitely block.
- To get around this, let’s add a timeout argument to our call to await().
    ~~~java 
        boolean completed = countDownLatch.await(3L, TimeUnit.SECONDS);
        assertThat(completed).isFalse();
    ~~~
- As we can see, the test will eventually time out and await() will return false.
- CountDownLatch Summary
    - When you create an object of CountDownLatch you pass an int to its constructor (the count), this is actually number of invited parties (threads) for an event.
    - The thread, which is dependent on other threads to start processing, waits on latch until every other thread has called count down. All threads, which are waiting on await() proceed together once count down reaches to zero.
    - countDown() method decrements the count and await() method blocks until count == 0
    - Once count reaches to zero, countdown latch cannot be used again, calling await() method on that latch will not stop any thread, but it will neither throw any exception.
    - One of the popular use of CountDownLatch is in testing concurrent code, by using this latch you can guarantee that multiple threads are firing request simultaneously or executing code at almost same time.
    - There is a similar concurrency utility called CyclicBarrier, which can also use to simulate this scenario but difference between CountDownLatch and CyclicBarrier is that you can reuse the cyclic barrier once the barrier is broken but you cannot reuse the CountDownLatch one count reaches to zero.
    - Java 7 also introduced a flexible alternative of CountDownLatch, known as Phaser. It also has number of unarrived party just like barrier and latch but that number is flexible. So its more like you will not block if one of the guest bunks the party.