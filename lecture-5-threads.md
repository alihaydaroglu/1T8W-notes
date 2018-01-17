#Lecture 5: Threads
Process model is based on two concepts:
* grouping of resources (program text and data, open files)
* execution - a "thread" of control (PC, SP, registers)
Threads use the same resources!

##Why do we need threads?
Sometimes a program needs to do multiple things concurrently, and it is much easier to implement with threads.
##States
**Thread Scheduler** multiplexes multiple threads on the same CPU. The threads can have **states**:
* Running
* Blocked
* Ready
* Exited

The state transitions are called:
* `yield()` goes from running to ready
* `sleep()` goes from running to blocked
* `wakeup()` uses another thread to wake up a thread, going from blocked to ready
* `thread_allocate()` and `thread_destroy()` create and kill states.
* `run()` goes from ready to run

##Process vs Threads
What actually is the motivation of threads? The intuitive example of *Microsoft Word*, where you have many processes doing backups, etc. We have an example to explain with.

### Web Server Example
Say you have a web server.

####One Single-threaded Process
You use a single threaded process. Why is it not a good design?
![5-web](/assets/5-web.JPG)
If you have a lot of requests coming in, you would have to handle all of these requests one at a time - can't handle them concurrently. For example, if you're waiting for I/O you just wait.

####Multiple Single-threaded Processes
If you have multiple processes, each of them handling a portion of a request it might be better?
![5-web2](/assets/5-web2.JPG)
In this case, the problem is each process is going to request the same file, so the same part of the disk is going to be read multiple times. This is kind of wasteful. Why not do it all at once? If you have a shared cache, why not let the processes share the same address space? But for the single-threaded processes, they have different address spaces.

####Multiple Threads
Now you have the same address space! This means low-cost communication via shared address space.

Also, thread creation, termination and switching *might* be faster. These threads are in a sense, **light-weighted processes** (this was the initial terminology for threads in the *Solaris* OS).

Multiple threads also allows overlapping computation and blocking (I/O) on a single CPU. This also allows you to take advantage of multi-processor systems.

So, back to the example, you have:
![5-web3](/assets/5-web3.JPG)
Now, a question:
* One thread per each client?   
  * Prevent a client from making too many concurrent requests
* One thread per each request?
  * When the demand surges, the overhead for switching between threads makes things worse in terms of performance.

####Thread Pool
So, might be a good idea to put a cap on the maximum number of threads we can have. Let's call it a **thread pool**, with a bunch of **worker threads**. We create these threads ahead of time, as opposed to on-demand. This eliminates the create-and-terminate process.

As requests come in, a **dispatcher** gives them to threads. This is going to balance a tradeoff between the two options talked about earlier.
![5-web4](/assets/5-web4.JPG)

####User Thread Model
It has a **thread table** that contains the states of each of the threads in a **thread scheduler**

The problem is, if you have multiple threads in this same process, how do you switch between them.

The thread scheduler is in the user space of this particular process, e.g. each process has its own thread scheduler. With threads, the OS treats the process only as a traditional process, threads don't need to be implemented in the OS.

![5-user](/assets/5-user.JPG)
(Ignore what is meant by the **kernel thread** we will look at it later. If this image is confusing, ignore it)

#####Advantages
This lets you have very very small overhead when switching between threads. It also allows each process to have its own scheduling policy.

The thread model is within the user space.

#####Disadvantages
This is not the best idea. Tons of problems.

Worst problem is **blocking system calls**. You can't go run other threads while one thread is waiting for I/O. While the thread is waiting for I/O, the whole process will become blocked, and all of the threads are blocked. This was one of the major advantage of threads, and we can't take advantage of it because the whole process is now blocked because of a single thread!

You could try to work around this using a special system call named `select()`, which is a non-blocking system call. It works for network sockets, but is only a patch. You could also change all the OS syscalls to non-blocking, but that defeats the purpose of using user threads and would require changing the kernel.

Also, if a **thread does not give up the CPU**, the thread scheduler cannot use timer interrupts so it cannot force them to switch.

###Kernel Threads
Instead of user threads, what if you have **Kernel Threads**. One user thread corresponds to one to one kernel thread.
![5-kernel](/assets/5-kernel.JPG)
####Advantage
No need to change blocking calls to nonblocking
####Disadvantage
The cost of switching or creating threads is significantly costlier, since it is now a system call, as opposed to a regular function call which was the case in the user thread model.

###Back to the Example
So, what if the **thread pool** model for the web server not the best option? Let's have as many threads as we have cores, and this way we can get rid of all the context switches.

If non-blocking system calls are available, we can design an asynchronous model.
* When a request comes in, the *only* thread responds
* If needed, a nonblocking I/O operation is started
* The thread records the state of the current request, and gets the next event
* The next even may be a new request, or a reply from the I/O subsystem

The single thread will essentially become an event handler.

This is an **asynchronous model**
* The sequential nature in previous designs is lost
* The state of computation is saved at every switch
* It is basically an event-driven FSM
* There is no need to use more threads

This requires **callback** mechanisms and **event notification** support from the OS kernel.
