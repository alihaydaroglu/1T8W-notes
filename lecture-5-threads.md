#Lecture 5: Threads
Process model is based on two concepts:
* grouping of resources (program text and data, open files)
* execution - a "thread" of control (PC, SP, registers)
Threads use the same resources!

###Why do we need threads?
Sometimes a program needs to do multiple things concurrently, and it is much easier to implement with threads.

**Thread Scheduler** multiplexes multiple threads on the same CPU. The threads can have **states**:
* Running
* Blocked
* Ready
* Exited

The state transitions are called:
* `yield()` goes from running to ready
* `sleep()` goes from running to blocked
* `wakeup()` uses another thread to wake up a thread, going from blocked to ready