+++
title = "Memory Virtualization"

date = 2022-02-07



[taxonomies]

categories = ["Operating Systems Design"]

tags = ["cs416"]

+++

Accessing physical memory can cause big issues if we do it directly. How can we virtualize the memory like the CPU so we can use it efficiently and safely?

<!-- more -->

Let's think of a super simple one process system. Our OS is located in the lowest part of memory, and everything after that is reserved for the user. Within an address space, we have the stack-heap structure discussed before. There's a big problem, that processes can access the memory that the OS is stored in and modify it. That's not good!

If we want multiple processes as well, we want to give processes memory that is managed by the operating system so they do not interfere with each other. Every process should have a self-contained *address space*.

We need dynamic allocation from memory as well so that our programs can become very powerful. The stack is not always sufficient for the programs we want to write. The dynamic heap is where all this memory management magic (say that five times fast) happens.

Memory accesses are built into `movl` instructions in the ISA. There are special operations to dereference the location stored in a register and access memory. These are known as *load/store* operations.

Memory accesses also happen from the code area by the instruction pointer, though that's typically not shown. If we want to count memory accesses, there is one fetch for every instruction, and another if the instruction is a load/store.

```nasm
0x10: movl 0x8(%rbp), %edi
0x13: addl $0x3, %edi
0x19: movl %edi, 0x8(%rbp)
```

In this example, there are 5 total memory accesses. There is one for every instruction, and each `movl` is a load/store because one of the operands dereferences a memory pointer.

Typically, we want to reduce this as much as possible because memory access is really slow, and we can't always rely on the cache.

So, what are some strategies to virtualize memory?

## Time Sharing

When we virtualized the CPU, we gave each process the illusion that it had its own CPU that were all running at the same time. The way this was done was through context switches that preserved CPU state in memory. We could do something similar by saving memory to disk when the process isn't running.

However, there is an immediate problem with this: it would be incredibly slow. Disk I/O even with SSDs is several orders of magnitude slower than DRAM, which is itself orders of magnitude slower than cache/register access. Considering how much processes play around with memory, this would be totally infeasible.

## Static Relocation

This is an interesting solution. The instructions stored as static data refer to specific memory locations that are hard coded in the application. However, what we could do is change those pointers and memory to memory that is currently available every time we load the process into memory.

For example, imagine the previous example which has instructions at `0x10`, `0x13`, etc. We could imagine that those memory locations are no longer available, so the OS changes the static code portion to `0x1010`, `0x1013`, etc. This means that all `jmp` and load/store instructions would also have to be rewritten as well since they directly refer to memory locations.

However, this translation is pretty expensive, although it is simple to implement. There are also security concerns, as always. There is no reason that the new memory couldn't be located somewhere that it can't go.

## Dynamic Relocation

We want the power of static relocation, but we need a way to manually protect each process from each other. This is such an important issue that it is usually provided as a hardware component: the **Memory Management Unit** (MMU). Whenever a process generates a virtual address in its address space, the MMU takes on the job for translating that address into a real address and giving it back to the process. This is a good example of modularization; we don't want to have the process to worry about memory, so we offload that job onto a specialized unit designed for that purpose. The MMU is managed only by the OS, so the user can never access it.

There are two general operating modes, as discussed previously: kernel space, and user space. Privileged kernel operations have full power over all of memory and the MMU, and users have to ask the OS for memory through virtual translation.

## MMU

The MMU contains a base register that stores the virtual address. It does the translation in steps:

1. is the memory in user mode? if so, you can use it.
