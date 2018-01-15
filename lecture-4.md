# Lecture 4: Processes

I missed this one, but the slides are pretty good.

## Multiprogramming

We run multiple programs concurrently because it increases CPU utilization. I/O intensive programs waiting for I/O, so use CPU for other stuff.

**Multiprogramming**: accommodating multiple processes in one physical address space. Each process can be I/O or CPU bound. Good to have a mix of both to maximize **CPU utilization**. A **long-term scheduler** decides which **job** to execute.

**Time-sharing** (**multitasking**): switching back and forth across processes very quickly - this is called a "**context switch**". The goal is to reduce latency when a user interacts with the computer.

### How does the CPU run Multiple Programs

