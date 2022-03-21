+++
title = "Swapping"

date = 2022-03-21



[taxonomies]

categories = ["Operating Systems Design"]

tags = ["cs416"]

+++

Sometimes we don't have a lot of memory to access. In those cases, we need a place to put our data. The natural solution is to put data in the disk, which has very high storage. This is known as **swapping**.

<!-- more -->

## Mechanism

The actual process of copying memory to disks is not complicated, it is simply file I/O and is supported by hardware. We swap memory out to disk, and swap in data from disk to memory. The complicated part comes when we introduce paging to this idea.

In order to accommodate this, we add a *present bit* to our page table entry. If the bit is 0, then the page has been swapped out to disk and the physical frame is no longer valid. When there is an attempted translation of a page table entry with an unset present bit, a trap will occur and the hardware will swap in memory from the disk, setting the present bit with a new translation for the new physical frame number.

An important note is that a process will *never* directly access the disk! Instead, a trap will swap disk and memory and the process will access the memory as dictated by hardware and the OS.

## AMAT

Swapping can be slow. We want to improve our **Average Memory Access Time (AMAT)**. A hit in this context is an access that goes directly to RAM, and a miss is a trap that will get the memory from disk into RAM.

$$AMAT = (Hit \\% \cdot T_m) + (Miss \\% \cdot T_d)$$

## Policies

When we swap in and out of memory, we need a way to find the most frequently accessed pages to reduce AMAT. There are several policies to accomplish this.

The optimal policy which has oracular knowledge of future page accesses can keep pages that will be accessed sooner and replace pages that will be accessed later. If we know that a page will be required soon, we're definitely not going to swap it out. Alternatively, if we know that it's going to be a while before the page is going to be accessed, we can swap it out, improving our AMAT. This is obviously infeasible when page access is random, which is almost all the time.

The simplest possible policy is, as with schedulers, **FIFO**. However, this does not work well. A better, also simple policy is **LRU**, or **Least Recently Used**. Most caches of this kind use LRU because of how effective it is despite its simplicity. The idea can be expressed in one line: swap out the page which has been least recently accessed. By keeping the pages of physical memory in a queue when they are created, it's simple to dequeue and swap out the page for a new, freshly enqueued page. The counter is reset every time that a page is swapped out.

LRU adds more memory usage in order to maintain the queue

> You would think that having more pages reduces the number of page misses. However, due to **Belady's Anomaly**, this is not always the case for FIFO. LRU does not suffer from this phenomenon

However, LRU does not take into consideration how many times a page has been accessed. A large I/O scan that is only used once might flush memory because it is recent without regard to how often it is used. When you stream a movie, you are sequentially accessing that data. This is a common access pattern in computer science.

We can use **pure LFU**, or **Least Frequently Used** to combat this. However, this runs into the opposite problem: what if a page was very frequently accessed in the past? The policy will not forget that so it won't be evicted despite it being very old.

A better approach is to combine both of the ideas of recency and frequency, through algorithms such as **LRU-K** or **2Q**. However, these algorithms can be expensive due to their increased complexity compared to these other simpler algorithms.

## Implementing LRU

There are two approaches: software and hardware, as always.

A software based LRU will be a sorted linked list. Accessing a linked list is slow, which means \\(O(n)\\) complexity for a memory reference, although replacing pages is very fast at \\(O(1)\\).
