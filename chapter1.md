# ECE353: Lecture 2

## Demos

From [2-intro-code](C:/Users/Ali/Google Drive/Engineering Science/1T8 W/ECE353/Lecture Notes - Clean/2-intro-code/intro).

### Demo 1 - mem.c and cpu.c

Run cpu.c using inline gcc command to demonstrate multiple instances can be run at the same time.

Run mem.c with and without the -no-pie option to demonstrate **Address Randomization**

### Demo 2 - threads.c

threads.c file demonstrates how to run multiple threads at the same time. Two threads incrementing the same value at the same time.

When running threads with numbers above a few hundred, the final value becomes not correct! The guilty code is here \(threadsv1.c\):

```c
void *worker(void *arg) {
for (int i = 0; i < loops; i++) {
    counter = counter + 1;
}

pthread_exit(NULL);
}
```

The line `counter = counter + 1`is converted into three instructions in into at least 3 instructions in gcc. These instructions are not guaranteed to be **atomic**, so they may be run in an interleaved way, causing a  **race condition**.

When dealing with race conditions, the result may or may not be correct. These conditions cannot be caught in a debugger, and they are notoriously hard to debug.

To fix the race condition, the following code uses **mutexes **to fix the issue in threadsv2.c

```c
void *worker(void *arg) {
for (int i = 0; i < loops; i++) {
	Pthread_mutex_lock(&m);
	counter++;
	Pthread_mutex_unlock(&m);
}
pthread_exit(NULL);
}
```

### Demo 3 - io.c

Here, running `gcc -o io io.c` and `./io` creates a 12 byte file using the`open()`** system call**.

To read the file:`cat /tmp/file`

To count the wordcount:`cat /tmp/file | wc`

To cat the file into another file, called **redirection**:`cat result.txt`

## Slides

### Virtualizing the CPU \(Chapter 6.1\)

Run one program for a bit, then run the next program for a little bit - but it's not as simple as it sounds.

Use **limited direct execution **to virtualize the CPU, and limit the overhead. This means the program runs directly on the CPU. However, the limit makes the program run with limited privileges.

The problems that require the limited execution are:

* How do you make sure the program doesn't do something it shouldn't be doing?

* How can the OS run itself while the user program is running? The OS is also software, a bunch of instructions

We need an OS to be in control, but we don't want to compromise the performance of the program.

### Brief Overview of the CPU

All CPU architectures have `load` and `store` instructions. Many instructions for arithmetic using registers.

Basics of a CPU include:

* **Program Counter**: Memory address of the next instruction

* **Instruction Register**: Current instruction

* **General Registers**: General purpose

* **ALU**: Arithmetic Unit

* **Stack Pointer**: holds the memory address of the_top of the stack_. The stack grows **downwards**, and grows every time a function is called, shrinks when a function is returned. Every funciton call is stored in a **stack frame**

* **PSW**: Program Status Word contains important bits

The CPU works with the following steps: **fetch**, **decode**, **execute**. In the BLITZ architecture, we have the same architecture

An intuitive solution to also being able to run the OS code, is to check whether every instruction is a special instruction, and switch to the OS kernel if it is. Otherwise, it will continue executing the normal program. However, the`if`statement that decides to switch to the OS is done in_hardware_.

CPUs have a **mode bit**in the PSW.:

* Set: **Kernel Mode**: executes any instruction \(also called **System Mode**\)

* Cleared: **User Mode**: executes a subset of instructions

The instructions that cannot be run in User Mode are the **Privileged Instructions**. I/O operations on the disk are an example.

### Processor Modes

OS Operates in **kernel mode**, and has all privileges to access devices and memory.

Applications operate in **user mode **with limited access to memory and access to I/O devices.

Now, the OS is in control!

Something about **kill **and **zombies **for programs that.

