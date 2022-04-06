+++
title = "Threads"

date = 2022-03-23



[taxonomies]

categories = ["Operating Systems Design"]

tags = ["cs416"]

+++

Concurrency is a powerful tool to increase performance in applications. This is usually accomplished using threads.

<!-- more -->

## Motivation

Clock frequency in CPUs has massively increased over the past few decades, growing at an exponential rate from the tens of MHz to GHz. However, circa 2005, clock frequency began to stagnate, and for the past decade the clock frequency of consumer CPUs have not increased past around 4 GHz. Why is that?

Clock frequency is typically increased by increasing the number of transistors in a single CPU. This can be done either by increasing the size of the CPU, or decreasing the size of the transistors. Because we don't want huge CPUs in our computers emitting tons of heat, we have opted to decrease the size of transistors. However, there is a limit to how small they can go. When a transistor starts getting smaller than 30 nm, power leakage will begin to affect the CPU, causing the same heat problems that a big CPU does.

In order to keep up with computing trends, new CPUs have multiple CPU cores with the same transistor density. However, utilizing multiple CPU cores requires parallelism in the CPU. An ideal CPU might run one process on each core, with each process getting full control over the CPU.

You can visualize this idea like a highway. Having multiple lanes allows cars to travel much faster than if there is only one lane. However, there can be issues when the lanes must combine, like in merging. This causes traffic like if there was only one lane, and can actually increase overhead as cars attempt to merge. Decreasing the amount of merging (or **synchronization**) is key to fast concurrency.

## Threads

We have discussed processes before; **threads** are not much different. They contain information about an execution such as the register values, open file streams, etc., but are more lightweight than processes. 

There are multiple frameworks for threading that are useful for different workflows. The **producer/consumer** model is typically used for data analysis. Multiple threads create some data which is handled by a synchronizing consumer thread. The **pipeline** model works like a real pipeline, where multiple threads are consumed in steps and pipeline flows. This is common in GPU programming. The **background** model will work similarly to the interactive/batch model in MLFQ. Background threads do behind-the-scenes dirty work when the CPU is idle, and foreground threads will run the actual important parts of a process.

Since threads exist within a single process, they share the same address space, including the stack and the heap. This can cause many, many bugs when it comes to accessing and modifying memory. We will discuss methods to solve these bugs later.

Threads do not share the same instruction pointer, however. Each thread can execute from different parts of the program. All register values are unique to each thread, as they can execute on any CPU.

Threads can technically share the same stack on the address space. However, this is pretty obviously a bad idea. Typically, threads will create their own stack so that each thread does not interfere with each other's local variables and execution, with their own stack pointer `%esp` to point to their stack.

## OS Support

It's all well and good to have threads, but we need a way for the OS to know how we're switching between threads. There are multiple ways to do this.

**Userspace threads** are a way to have concurrency regardless of what the OS thinks. A run-time will execute along with your program and swap between threads itself without the OS knowing. This is how `pthread` works, and also how Project 2 does. However, the power that concurrency grants for multicore CPUs is still dependent on the number of kernel threads generated, because those are the only ones that run on each CPU.

**Kernel threads** can be associated one-to-one with user-level threads. This way, threads can take advantage of multiprocessing because the kernel will parallelize operations across CPUs. However, this incurs an overhead for the syscalls needed to parallelize threads, and there will never be enough kernel threads for the user threads.

## Scheduling

Small operations like incrementing a variable can be difficult when we have threads accessing the same data. The simple instruction `balance++` where `balance` is stored at memory location `0x9cd4` would have this assembly output:

```nasm
0x195    mov 0x9cd4, %eax
0x19a    add $0x1, %eax
0x19d    mov %eax, 0x9cd4
```

If we have a context switch when `%eip` is at `0x19a`, the first thread will `add` but not store the new value back in memory. That means that the variable `balance` would be less than expected because the `add` by the first thread would not be stored into memory.

This can lead to *non-determinism*, where the same input will cause different outputs, as well as *race conditions*, where the result of a program will depend on the CPU timing of different operations.

We want these three instructions to be executed uninterrupted. This is known as **atomicity**. In this **critical section**, no process or thread can interrupt the currently executing thread. However, if we allow programs to do this, then they can take advantage of this property and essentially disable the timer for context switches by declaring the entirety of execution atomic. We can't have full blocking on interrupts for threads, so we only lock other threads from executing at the critical section. When the timer interrupt hits, the scheduler will still take control, but no other thread will be able to execute at the critical section.

## Synchronization Primitives

There are many high-level **synchronization primitives** in the OS that exist to ensure the correct order of instructions. Each kind of lock is designed to solve a specific problem; no single lock can solve all of them.

We want to have correctness in our concurrency. This means that we must have mutual exclusion where only one thread can access the critical section at a time. If multiple threads are waiting for something, they cannot be stuck forever, they must make progress. Similarly, threads cannot be forced to wait an unreasonably large amount of time. Concurrency must also be fair and not favor any thread over any thread. And obviously, we do not want to overly tax the CPU with the overhead of synchronization.

We need underlying hardware atomic operations in order to implement synchronization. When these hardware instructions execute, *no instruction or interrupt* can execute. Example instructions are `Test&Set` and `Compare&Swap`.

The reason we need this is because if we don't make these instructions atomic, we can encounter race conditions. Imagine an unset lock with a while loop and two threads. The first thread successfully enters the loop because the lock is unacquired. However, before it can acquire the lock, the thread switches and the second thread executes and acquires the lock. Now the first thread has not acquired the lock but it is still in the loop, so when it runs again it will acquire the lock even though it has already been acquired, allowing two threads in the same critical section! In order to combat this, testing and setting are the same atomic operation so there isn't a gap that can allow a switch in between those two operations.

## Spinlocks

Making our implementation fair is not as easy. If we imagine the most basic spinlock which switches back and forth, a program that knows how long a context switch will take will acquire the lock right before every context switch so no other program will be obtain it.

In order to introduce fairness, the ticket system for schedulers can be used here. Every thread is assigned a ticket for every turn it gets, with the first thread getting ticket 1, second ticket 2, etc. Now when a thread releases a lock, the ticket increments to the next thread. Even if it wants to maliciously acquire the lock before a context switch occurs, it can't because it has lost its turn.

We need a special atomic function to make this work like `Test&Set` with basic mutex. We should be able to get a ticket number and increment it atomically. The function used for this is `Fetch&Add`.

```c
int FetchAndAdd(int *ptr) {
    int old = *ptr; // these two lines are
    *ptr = old + 1; // executed atomically!
    return old;
}
```

Spinlocks are fast when we have a short critical section because we're not context switching very often, so when we switch between locks constantly we want to avoid that overhead as much as possible as that dominates execution time. However, in a situation where there is only one CPU, letting the threads spin on a lock is extremely wasteful. The CPU scheduler has no idea that a thread is waiting for a lock so it will ignorantly run it even though all it does is spin.

One alternative is to yield the thread instead of letting it spin, telling the CPU scheduler that the thread doesn't need to run right now. *In general*, a shorter critical section is okay to spin, but a longer critical section is not okay to spin.

## Condition Variables

Mutex locks are just a way of restricting access to a critical section. It's synchronous, but there's no order to the way that the threads have to run. That is up to the caprices of the scheduler. However, let's say that ordering is significant to the execution of our threads. How can we ensure that threads are executed in a certain order?

We need some functions and concepts to implement this. There are two system call functions `wait(cond_t, mutex_t)` and `signal(cond_t)` that are used with **condition variables** `cond_t`.

Let's assume that a thread has acquired the lock, but it has not satisfied the condition to run yet because another thread must run first to preserve ordering. In that case, the condition variable will indicate that the thread cannot run and it will call `wait` to sleep the thread until the condition variable changes, releasing the lock so that the other thread can run. The caller of `signal` wants to wake up that sleeping thread, unless there are no sleeping threads, in which case it does nothing.

The best time to use condition variables is in a producer/consumer system, like in a pipe. Producers write to a pipe, and consumers read from the pipe. The pipe buffer has a limited size, so the producer can only add data to it if the buffer is empty, and the consumer can only read from it if the buffer has contents. For simplicity, assume a buffer of single unit size for now.

```c
void *producer(void *arg) {
    for (Int i = 0; i < loops; i++) {
        Mutex_lock(&m); // lock the mutex
        while (numfull == max) { // while the buffer is full,
            Cond_wait(&cond, &m;) // wait
        }
        do_fill(i); // put data in buffer
        Cond_signal(&cond); // signal conditional variable
        Mutex_unlock(&m); // unlock the mutex
    }
}

void *consumer(void *arg) {
    while(1) { // consume on and on forever
        Mutex_lock(&m); // lock the mutex
        while (numfull == 0) { // while buffer is empty,
            Cond_wait(&cond, &m); // wait
        }
        int tmp = do_get(); // get the data
        Cond_signal(&cond); // signal conditional variable
        Mutex_unlock(&m); // unlock the mutex
        printf("%d\n", tmp); // do something with data
    }
}
```

Let's consider an example with a producer and two consumers, as well as a FIFO scheduler. The first consumer thread runs and waits immediately because the buffer is empty. The second consumer thread does the same. The producer thread runs through its entire loop, filling the buffer, then when it loops again it will wait (since our buffer is size 1). The first consumer thread will return to the while loop, and now that the buffer is no longer empty it will fetch the data, signal the thread, and finish.

However, there is a problem here. The `Cond_signal` function signal is extremely generic and does not signal a specific thread. Instead, it will signal a random thread, which may be the producer or other consumer thread. But the whole point of using conditional variables is so that we have ordering!

We need to have *multiple* conditional variables:

```c,hl_lines=5
void *producer(void *arg) {
    for (Int i = 0; i < loops; i++) {
        Mutex_lock(&m); // lock the mutex
        while (numfull == max) { // while the buffer is full,
            Cond_wait(&empty, &m;) // wait until buffer is empty
        }
        do_fill(i); // put data in buffer
        Cond_signal(&fill); // signal to show buffer is somewhat filled
        Mutex_unlock(&m); // unlock the mutex
    }
}

void *consumer(void *arg) {
    while(1) { // consume on and on forever
        Mutex_lock(&m); // lock the mutex
        while (numfull == 0) { // while buffer is empty,
            Cond_wait(&fill, &m); // wait until buffer is somewhat filled
        }
        int tmp = do_get(); // get the data
        Cond_signal(&empty); // signal to show that buffer is empty
        Mutex_unlock(&m); // unlock the mutex
        printf("%d\n", tmp); // do something with data
    }
}
```

In this example, the consumer thread will only run when the producer has signaled that there is content in the buffer.

## Semaphore

Conditional variables do not have any state. They only signal the condition to wake or sleep a thread. This can be useful if you have lots of threads that do the exact same thing, like in a producer-consumer model. However, if you want a more complex way to represent state, you need a **semaphore**.

Semaphores are essentially conditional variables that contain an integer value which can be changed by threads. This allows threads to have multiple options based on a single variable shared across multiple threads just by changing the value in the semaphore. The semaphore is incremented every time a thread is woken, and decremented every time a thread sleeps.

The atomic operations for semaphores are `Allocate&Initialize`, `Wait`, and `Post`. We must first create the semaphore by allocating/initializing it. The `Wait` operation decrements the semaphore if it is greater than 0, and `Post` will increment the semaphore and wake a sleeping thread.

In this context, joining and exiting threads have no complication with mutexes; all you need to do is wait and post the semaphore, respectively.

We can actually construct locks using semaphores. Acquiring and releasing locks work the same way as waiting and posting, so you can just use those same atomic operations instead of `Test&Set`. In reverse, you can build a semaphore using locks and conditional variables, though in practice that's a little more complicated.
