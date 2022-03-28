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

Making our implementation fair is not as easy. If we imagine the most basic spinlock which

### Mutex

Mutex stands for **mutual exclusion**. Mutexes are initialized, then acquired in order to enter the critical section. If the lock cannot be acquired, then the thread must wait. This waiting can either take the form of **spin** or **block**. Spinning simply keeps the thread in a while loop until the mutex is released, whereas blocking sets the thread to a block state which will be checked by the scheduler. After the mutex is released by the acquiring thread, other threads can acquire the thread and lock it in the same way until the mutex is destroyed.
