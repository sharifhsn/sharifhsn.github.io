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

## Address Translation

To start out with, we will make some assumptions about how address spaces are laid out. They are all contigous spaces in memory of the same size that are smaller than physical memory. As we will see later with segmentation, this assumption will not hold up, but it useful for now.

In order to translate a *virtual address* to a *physical address* in memory, we need some kind of reference space in memory. This is given by the **base** and **bounds** registers, which give the physical memory locations of where the virtual address space starts and ends, respectively. These registers are not part of the regular ISA and are instead part of the previously mentioned MMU, and operating on them requires privilege. The MMU can also throw an exception upon illegal memory access which is handled by our OS.

Our OS can manage memory in this simple way through the mechanism of a *free list*, which is a linked list which indicates free spaces, which is also used for `malloc`.

As discussed before, the address space has a structure where the stack grows downward and the heap grows upward. However, if you look at a diagram of this for more than five seconds, you might notice a giant chasm between the stack and heap. Relying on our assumption of a contiguous space of memory, this is a lot of wasted memory. Some address spaces fix this problem through **segmentation**.

## Segmentation

As the name implies, a segmented address space is split into multiple segments with its own base/bounds pair to indicate its logical beginning and end. We can split every piece of the address space into its own segment, so stack goes in one segment, heap goes in another, etc. This way, although the virtual address space has this nice alignment, we are not wasting physical memory with our *sparse* address space.

In order to translate a virtual address, we treat the address as an offset with a segment. For example, if we were trying to access a virtual address in the heap, this would be the formula:

$$physicalAddress = physicalBase + (virtualAddress - virtualBase)$$

Before we allow this translation to take place, however, we must check that the value does not exceed the physical bound. If this is the case, then we have caused the infamous **segmentation fault**.

A faster way to do with this masks is through bitwise operations. The top two bits of the address refer to the segment, either the code, stack, or heap. The rest of the address is the offset within the segment. This way, we can perform the bounds check before accessing the physical address by checking if the expression in the parentheses exceeds the virtual bounds.

However, one issue with this means that each segment gets the same maximum size, which is \\(2^{offset}\\). If we want a bigger heap, we're out of luck. We can solve this by tracking instructions instead of bits. For example, an instruction fetch for `%rip` will come from the code segment, so we don't need to put that in the address that it is in the code segment.

Another issue is that the stack segment grows downwards instead of forwards, which means that its segment works differently. We need an extra bit for segments that indicates whether they grow forwards or backwards, which will be set to 0 for the stack and 1 for everything else.

If that bit is false, then we subtract the offset from the base instead of adding it as above.

## Sharing :)

Sometimes, processes share the same code. It seems wasteful to copy the same code segment every time we have a new process, so modern operating systems implement **sharing** for code segments. Sharing can be implemented for any segment, but it is most common for code since it's read-only. This is important for dynamically linked libraries since many processes will access the same library, like `libc`.

Hardware adds extra *protection bits* to each segment indicating its `rwx` value similar to a file. If a segment is indicated to be `r--`, then the OS can secretly share the segment between multiple processes, assured that they will only read it. This concept of a read guard allowing for multiple shared references as opposed to a write guard which only allows for one mutable reference will become *very* important when we discuss concurrency.

We have only been working on a few segments so far, but segmentation could theoretically be extended to as many segments as you want. In order to have *fine-grained* segments, a segment table is needed to quickly access thousands of segments.

There are a few more things that the OS needs to do in order to support segmentation. One is that it must preserve all base-bounds pairs upon a context switch, since the location of the address space is now more complicated. Another is that the growing and shrinking of segments must be managed through `sbrk`-like system calls that shift the bounds of the heap/stack. The final and most important issues that allocating variable size address spaces runs into the same problems of `malloc` where external fragmentation is difficult to avoid. Like with `malloc`, there is no perfect solution, ranging from algorithmic free lists to compact memory which rearranges segments every time a new one is created.
