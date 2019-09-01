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