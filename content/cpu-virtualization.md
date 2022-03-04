+++
title = "CPU Virtualization"

date = 2022-01-04



[taxonomies]

categories = ["Operating Systems Design"]

tags = ["cs416"]

+++

We need a way to map on to the physical CPU through our operating system. We do this through virtualization.

<!-- more -->

## Processes

A **process** is an *execution stream* in the context of a process state. It is a self-contained stream of executing instructions on the CPU. Each process has a state which is composed by everything that the code can affect or be affected by, for example *registers*, address space, open files, etc. We need this state so that we can pause and resume processes without resetting the whole thing, just by picking up where the state left off.

You can think of address space as the region of memory in which the CPU operates, though this is a simplification due to memory virtualization.

A process is technically different than a program. When we talk about a program, it's usually a collection of files on our computer. The process refers to the actually executing code which is dynamic with respect to its code. We can have multiple processes executed that are the same program.

Processes do not share information with other processes. This is distinct from the idea of threads. If a process examines the "same" memory address with respect to its address space as another process, they will see different values. Even though both process are looking at `0xFFE84264` the memory virtualization means that they are looking at different physical memory. Threads share data so they have the same address space; they are in a way a lightweight process.

## Address Space

As mentioned, every process has its own address space. The OS assigns a chunk of memory towards it (again, this memory is virtual). At the high address, there is a stack of local variables that grows downwards. This stack contains the execution of the program, such as functions being called. The code segment which is the actual program is read from the absolute bottom. There are a few segments like `.data` and `.bss` which refer to global initialized/uninitialized variables, respectively. Then there is the heap, which is dynamic and grows upward. Usually, the heap is much larger than the stack because it contains allocated memory for the process.

> The stack and the heap grow towards each other, so you might think they would collide at some point. In reality, there is a wide enough chasm between them that if they collide, you have bigger problems to worry about.

## CPU Virtualization

Our problem is that a CPU can only execute one execution stream at one time. We want processes to be running at the same time, so how do we achieve the illusion that the process has full control of the CPU?

One solution is *direct execution*. We will run the process as simply as possible and execute all of its instructions in sequence. This is extremely simple to implement, but it means we can only run one process at once. If the process runs forever, then the CPU is stuck. Processes might write to other data that it's not supposed to, or do some slow I/O operation, or execute privileged instructions it's not allowed to accesss. In general, we don't want to trust processes to behave. Operating systems use *limited direct execution*, which maintains some control over the execution of the process instead of giving it unfettered access to the CPU.

## System Calls

We want to make sure user process can't harm other processes. CPU hardware supports privilege levels for security reasons. These instructions should only be run in kernel space by the operating system, and user processes in user space can not access them. Privileges can have multiple levels. In order for processes to access privileges, we use **system calls** that can either pass the function off to the OS or give the process privilege, also known as a *trap*.

```c
ID1 = syscall(SYS_getpid); // syscall

ID2 = getpid(); // call to libc
```

Let's say a process wants to execute `read()`, which is a syscall. It will move the syscall ID for read `0x6` into the `%eax` register for execution, then run the instruction `syscall`.  The operating system will then read the trap table to figure out what to do. The trap table is located in the hardware, which typically has an entry for system calls. Other traps could be `illegal access` for memory region that it doesn't have access to. A trap just refers to a hardware operation that triggers on some software interaction. This is why Javascript can't just randomly hack your computer from the internet; it is executing within a user process e.g. Chrome and is limited by the hardware traps in what it can do.

## Multiprogramming

We want to make sure that multiple processes can run at once. This means we must switch between processes. There are two components to this; how to switch, and when to switch.

The dispatch loop has a simple structure, but the devil is in the details.

```c
while (1) {
    // run process A for some time-slice
    // stop process A and save its context
    // CONTEXT SWITCH
    // load context of another process B
}
```

We have multiple ways to context switch. One way is *cooperative multi-tasking*. We trust the process to give up the CPU when it has judged some amount of work has been done or time has passed. We will provide `yield()` syscall for magnanimously donating CPU time to another process. However, this is annoying to program in, so most programs do not do this.

Modern programs use *true multi-tasking*, which gives OS control over which processes are running. The hardware will generate a timer interrupt every 10 ms, for example (Linux) that the OS will decide when to switch or not.

## PCB

The **process control block** is a descriptor of a process that saves its context into a `struct`. There is a `struct` for every process that is running in the CPU. A PCB stores the following information:

- PID -  unique identification for a process

- Process state (`enum` running, ready, or blocked)

- Execution state (registers at time of pause)

- Scheduling priority

- Accounting information (parent/child procs)

- Credentials (resource access/owner)

- File pointers

When we save the context of the process in the dispatch loop, the PCB is where it is saved. It is typically less than a few kilobytes depending on the complexity of the process. The PCB is stored on the *kernel stack* which is a stack data structure located in kernel space. Switching a context means moving the stack pointer to the process that is going to resume and then retrieving that PCB from the stack. Then, we exit kernel space and resume executing B in user space by restoring the PCB information to the CPU.

When a process is doing I/O, it's not using the CPU. This is an excellent point where the OS can block the process and run some other processes that actually need the CPU. Once the I/O is done, it is moved from the blocked state to the ready state. The convention is that only ready state programs can go to the running state.

## Process Creation

One way to create a process is just creating one from scratch. We will load the code we're running into memory at the bottom of the stack and create the stack. We also create a PCB with a kernel stack and everything, then put the process in the ready state. However, there are lots of complicated parts of processes that are difficult to initialize from scratch e.g. permissions, I/O, environment vars so this is generally not preferred.

A better way is to clone an existing process and change it to the appropriate information. `fork()` clones the calling process and `exec(char *file)` replaces the current process with the new process.

When we fork, we save the current state of the program as a new PCB that is added to the kernel stack, which is very optimized to use copy-on-write semantics.

> The basic idea of copy-on-write is that it uses a pointer/reference for memory until we need to modify memory. This is an incremental process that trades off some overhead for a pay-as-you-go memory saving system.
