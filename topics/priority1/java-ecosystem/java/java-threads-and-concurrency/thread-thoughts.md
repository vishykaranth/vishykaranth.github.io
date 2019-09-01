---
layout: page
title: Thread Thoughts
permalink: /thread-thoughts/
---

### CountDownLatch v/s CyclicBarrier
- The key difference is that CountDownLatch separates threads into waiters and arrivers 
- while all threads using a CyclicBarrier perform both roles.

### Real Life examples 
- With a latch, the waiters wait for the last arriving thread to arrive, but those arriving threads don't do any waiting themselves.
- With a barrier, all threads arrive and then wait for the last to arrive.
- Your latch example implies that all ten people must wait to lift the stone together. 
    - This is not the case. 
    - A better real world example would be an exam prompter who waits patiently for each student to hand in their test. 
    - Students don't wait once they complete their exams and are free to leave. 
    - Once the last student hands in the exam (or the time limit expires), the prompter stops waiting and leaves with the tests.

### Hypothetical theater:
- It is called Mutex if only one person is allowed to watch the play.
- It is called Semaphore if N number of people are allowed to watch the play. 
    - If anybody leaves the Theater during the play then other person can be allowed to watch the play.
- It is called CountDownLatch if no one is allowed to enter until every person vacates the theater. 
    - Here each person has free will to leave the theater.
- It is called CyclicBarrier if the play will not start until every person enters the theater. 
    - Here a showman can not start the show until all the persons enter and grab the seat. 
    - Once the play is finished the same barrier will be applied for next show.
- Here, a person is a thread, a play is a resource.   

### Real World Example 
- **CountDownLatch** 
    - A Multithreaded download manager. 
    - The download manager will start multiple threads to download each part of the file simultaneously.
    - (Provided the server supports multiple threads to download). 
    - Here each thread will call a countdown method of an instantiated latch. 
    - After all the threads have finished execution, the thread associated with the countdown latch will integrate the parts found in the different pieces together into one file
- **CyclicBarrier** 
    - Same scenario as above.
    - But assume the files are downloaded from P2P. 
    - Again multiple threads downloading the pieces. 
    - But here, suppose that you want the integrity check for the downloaded pieces to be done after a particular time interval. 
    - Here cyclic barrier plays an important role. 
    - After each time interval, each thread will wait at the barrier so that thread associated with CyclicBarrier can do the integrity check. 
    - This integrity check can be done multiple times thanks to CyclicBarrier 
    
    
    
### References 
- https://stackoverflow.com/questions/10156191/real-life-examples-for-countdownlatch-and-cyclicbarrier
    