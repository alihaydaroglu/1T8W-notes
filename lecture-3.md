# Lecture 3: OS - a bird's eye view

Two problems from previous lecture:

* How to make sure a program isn't doing something it's not supposed to be doing?
* How to make sure we get to run the OS code when needed?

Distinction between **User mode** and **Kernel mode** - review last lecture's notes.

How can the applications enter the OS kernel? Can an application just set PC to the address of an OS instruction by jumping?

### How do we enforce that only the OS operates in the kernel mode?

Applications running in user mode need to perform a **system call**. To perform a system call, a user program needs to execute a special instruction called a **trap** or **syscall**.

All the **trap** does is jump into the kernel while simultaneously raising the privilege level to kernel mode. The application calls a library procedure that includes the appropriate trap instruction. Fetch/Decode/Execute begins at a pre-specified OS kernel entry point for a system call. CPU is now in kernel mode.

All the instructions that run after calling the **trap** are the OS instructions on kernel mode.

When the system call returns, we need to go back to user mode using a **return-from-trap** instruction, typically called ```reti```. 

The OS needs to cooperate with the CPU to execute this trap.

The **Execution Context/Environment** changes after the system call is made. This is includes saving the caller's registers, PC, and PSW bits to ensure that all of these are saved and restored and correctly after the return-from-trap instruction. 

Different CPUs use different places to save them - Intel x86 uses a **kernel stack** to save all of them. It is a per-process kernel-stack, meaning that every single user process will have its own kernel stack inside the kernel. Return-from-trap instruction will pop the values off

### How does the trap instruction know which code to run?

There are many system calls, but only one ```trap``` instruction. One caveat is that we don't want any user program to know the starting address of these system functions, so we want to have a way of doing it indirectly. We want to add an additional **level of indirection**. 

To achieve this indirection, we use a **system call number** instead of the explicit address of the function. To store them, the kernel sets up a **trap table** at boot in kernel mode, and lets the hardware know where it is. A better name for this table is the **interrupt table** since it only contains one address for the trap (there is only one address that is taken for *all* of the system calls).

So, a system call makes the hardware go into the trap table, which has one entry that points to some function that determines what system call is to be executed, something like this:

``` python
def trap_handler(int syscall_number):
    switch(syscall_number):
        case 1: ...
        ...
        case n: ...
```

Each case corresponds to a different system call, and each of those might use the privileged instructions if necessary (since they run in kernel mode). There are a few system calls that don't use privileged instructions, but those use information or methods that processes aren't capable of - for example killing other processes.

The trap table is also called the interrupt table, and it has entries for hardware **interrupts** and **exceptions**. Interrupts are hardware, exceptions are software.

The syscall number is stored on a stack or a registers.

### Interrupts and Exceptions

Interrupts are **asynchronous**.

Check lecture slides for a few diagrams I missed.

How does the hardware know where the Interrupt Table is located? Hardware knows.

####PSW in BLITZ

Called the **Status Register** in BLITS. Three condition codes, Z, V and N. Privileged bits:

* I: interrupt enable
* User/kernel mode
* Paging

#### Handling Interrupts

The interrupt types are: Hardware (asynchronous), and exceptions (synchronous)

There 16 user registers and 16 kernel registers

Refer to lecture slides for full protocol on how stuff gets saved and exactly what happens when entering the interrupt (PC, PSW pushed to system stack, PC points to stuff.)

#### Returning from a trap handler

`reti` instruction in Blitz does the following:

1. Restore PC by popping top value from the system stack into PC
2. Restore Status Register by popping from stack into status register
3. Pop and discard next value from system stack (**Exception Info Word**)

None of the users are affected!

## When a program is running, how can the OS run?

We could trust programs to behave and wait for a `yield()` syscall which transfers control to the OS.

This is the **cooperative** approach originally used on Mac OS and Windows. This is dumb! A bad program can take over the CPU forever.

Without **cooperation**, the OS depends on **timer interrupts**.