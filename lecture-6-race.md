#Race Conditions
Let's get started with an example.

##Communication across two threads
Let's say two threads share an address space, and they use a shared, bounded (finite amount of space) buffer.

![6-buff](/assets/6-buff.JPG)

Let's say the OS has the following interfaces:
* `send(message)`: if there is room in the buffer, insert into buffer; else, wait until there is room
* `receive`: returns from buffer if nonempty

Assume there is *one CPU per thread*, where the processors are connected to a bus where there are I/O devices and memory.

###Produced-Consumer Problem
Very common problem. One thread is always the sender, other is always receiver. The producer needs to add messages to the buffer before the consumer removes it, producer needs to wait for the buffer to be non-full before adding more to it.

Here is a simple solution that works:
```C
message_t buffer[n]
int in = 0, out = 0;

send(message_t msg)
    while in - out == N
        do nothing
    buffer[in % N] = msg
    in = in + 1

message_t receive()
    while in == out
        do nothing
    msg = buffer[out % N]
    out = out + 1
    return msg
```
The assumptions on this is the **one writer principle**. Only one thread updates each shared variable. Only receiver updates `out`, only sender updates `in`.

Each sender has its own processor

###Introduce a Race Condition

**What if we allow several senders?**. This introduces a **race condition**, where the result depends on the exact timing of two threads.

Another type of race condition that is hard to debug is in the line `in = in + 1`, which in assembly becomes a possible hazard:
```
LOAD in, R0
ADD R0, 1
STORE R0, in
```
It is very hard to reproduce, so it is extremely hard to debug.


###Fixing Race Conditions
We want to systematically prevent this interleaved execution which leads to a race condition. Let's make threads coordinate with each other so they are **mutually excluded** in accessing **critical sections** of code. We also call these sections **atomic actions**. We need a **lock** to these actions.

Let's use some primitives to work with these locks: `acquire()` and `release()`. While the thread is holding the lock, no other threads who attempt to enter a critical section cannot do so.

Be careful when using these primitives, because you might mess it up where you have a dead lock!

###Implementing ```acquire()``` and ```release()```
Try this:
```C
struct lock
    int state

acquire(lock L)
    while L.state == LOCKED
        do nothing
    L.state = LOCKED

release(lock L)
    L.state = UNLOCKED
```
This doesn't work! there is a race condition since we have two writers to the shared state variable. This might fail and let both into the locked part at the same time.

So, did we go back to the beginning? Not really, we reduced the general problem to a narrower problem, which is the problem of checking and updating a shared lock. If we make the check-and-update-lock atomic, then we fix it.

What if we **disable timer interrupts**? The thread scheduler cannot run, and no other thread can run on this processor. This is not a good idea.

The solution is in **hardware support**. We have a **Test and Set Lock (TSL)** instruction from the hardware.
```
TSL(LOCK)
    do atomic
        RX = LOCK
        LOCK = LOCKED
    RETURN RX
```
There are variants on this, but it's always about making these two steps atomic.
