# Lecture 4: Processes

I missed this one, but the slides are pretty good.

## Multiprogramming

We run multiple programs concurrently because it increases CPU utilization. I/O intensive programs waiting for I/O, so use CPU for other stuff.

**Multiprogramming**: accommodating multiple processes in one physical address space. Each process can be I/O or CPU bound. Good to have a mix of both to maximize **CPU utilization**. A **long-term scheduler** decides which **job** to execute.

**Time-sharing** \(**multitasking**\): switching back and forth across processes very quickly - this is called a "**context switch**". The goal is to reduce latency when a user interacts with the computer.

### How does the CPU run Multiple Programs?

The CPU has a Memory Management Unit \(**MMU**\), mapping citrual addresses to physical addresses. A simple one does **relocation** of virtual to physical addresses and **memory protection** by keeping references within bounds. First is done with a **base register**, the latter with a **limit register**. Each program has its own **base** and **limit** registers.

There are **Page Table Register** which are used when paging is enabled to allow translation by MMU from virtual to physical addresses. To be discussed in a few weeks.

## The Process Abstraction

**Program** is a passive entity, usually stored in an executable file in a file system, containing instructions and static data values.

**Process** is a program in execution, or a running program.

### What constitutes a process?

We need to understand its execution context, and what it can read or update. One obvious component is memory: the memory that a process can address is called its **address space**. Other things include registers, PC, SP, I/O info, etc.

#### Address Space

Set of memory sections accessible to a process. Includes

* Text \(program code\)
* Stack \(frames\)
* Data \(globals and constants\)
* Heap \(Dynamically allocated memory\)

The virtual address space of a process:

|  |
| :--- |
| Stack |
| **Gap** |
| Heap |
| Data |
| Text |

Stack and Heap eat into the Gap, until the process is out of memory.

### Why use the Process Abstraction?

Allows multiple programs executing in the same physical address space. **Virtualizes the CPU**.

## Process States

A process is **running** until either

* it is _descheduled_, in which case it becomes **ready**,
* it _initiates I/O_, in which case it becomes **blocked** while waiting for the result of the I/O. 

A **blocked** process becomes **ready** upon the _I/O completion_, and the **ready** process starts **running** when it is _scheduled_.

### Process Control Block \(PCB\)

This is used for saving states in a context switch.

The OS needs to keep track of the following information about a process:

* State: blocked/ready/running/_zombie_
* Program counter
* CPU registers
* CPU scheduling information
* Memory management information: base and limit registers or page tables
* Accounting information
* I/O information: a list of open files

State is saved into PCB upon descheduling, and reloaded from PCB upon scheduling.

The time it takes to perform a **context switch** is called **overhead**, and we want to minimize it.

## Syscalls related to process abstraction

**Process creation** requires loading code and static data of a program from the disk into the memory, and creating the address space of the process \(including code, static data, heap and stack\) in the physical memory. You could load it in two ways:

* **lazily**: load it up page by page, in chunks, not all at once
* **eagerly**: load it up all at once

Some useful syscalls:

* All processes have unique process IDs, retrieved with the `getpid()` syscall.
* `fork()` syscall copies a process \(parent and child\) and returns both.
* `exec()` replaces an address space with a new program. 
* `exit()` or `kill()` are used for process termination.



