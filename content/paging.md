+++
title = "Paging"

date = 2022-02-20



[taxonomies]

categories = ["Operating Systems Design"]

tags = ["cs416"]

+++

The main problem that segments have introduced to managing memory space is that their variable size wastes memory through fragmentation. Fixed-size pieces that are easier to handle are much more popular: these are known as **pages**.

<!-- more -->

## Pages

Address spaces are split up into multiple pages, typically a power of 2. For example, let's picture a tiny 6-bit address space that can only address 64 bytes. We could split it up into four 16-byte pages, that we'll refer to in sequence.

The physical representation of these pages is through **page frames**, which are sequenced directly in memory and are ordered. The pages in the virtual address space map directly to page frames in physical memory, with no respect for order.

Memory management of the free space here can be done with a simple free list. All it needs to look for is four free page frames *somewhere* in memory and map each page to a page frame. This mapping is stored in a **page table** which is kept *per process*, since every process has its own address space.

## Page Translation (IMPORTANT)

In our example from earlier, we worked with a 6-bit address space. The highest order bits are reserved for the virtual page number, and the lower bits are reserved for the offset within the page. For example, the virtual address 21 would be `010101` in binary. The top two bits `01` tell us that we are looking for virtual page 1. The OS checks the page table and sees that VP2 maps to physical frame 7, which is `111` in binary. The OS then translates the physical address by replacing the VP bits with the PF bits. In this example, the new address would be `1110101`, or the 117th byte in memory.

## Page Tables

We've talked about page tables a bit, but let's go into details. They are a data structure like any other, but they can get very large. Each mapping in the table, which is called a **page table entry (PTE)**, is typically \\(2^2\\) bytes in size. It's often simpler to think of these calculations in terms of the bits involved, since they will always be a power of two. With that in mind, this is the formula for page table size:

$$ pageTableSize = VPN + PTE $$

That's 22 bits for a typical 32-bit system, which is pretty massive at 4MB per page table, which is again per process. We obviously can't keep this in the MMU, so we need to actually store it in memory.

How do we organize the page table as a data structure? The most obvious way is as a linear table, aka an array. The VPN is an index in the array, and the value at the index is the PTE which gets the PFN. The PTE itself contains the PFN plus some helpful bits for us. There is a valid bit checks if the mapping even exists, protection bits for privileged memory, and others.

This access is still extremely slow for load store operations. We need a better solution!

## TLB

The hardware comes to the rescue here. MMUs provide a **translation-lookaside buffer** to cache commonly used translation. When a load-store instruction is executed, the CPU will first check the TLB if there exists a translation for the specified address if it exists. If it does, then it can quickly perform the operation without performance overhead of going back and forth on memory so often. If it doesn't, the CPU hands the reins over to the OS to do its own page and offset translation and it will cache the translation after the OS gives it. Here's the basic steps:

- extract VPN from virtual address

- check if VPN is in TLB

- if hit, then perform the PFN concatenation and access the memory

- if miss, hardware will check the page table to find the translation

- update the TLB with the translation

- retry TLB, now a guaranteed hit

All of this assumes that memory is valid, accessible, and unprivileged; these can cause exceptions that the OS will handle.

Step 4 is expensive and it is therefore the step that we want to avoid as much as possible. Luckily, many memory accesses reside on the same page. For example, arrays are almost guaranteed to have the same TLB hit because they are organized contiguously. **Spatial locality** helps us increase our hit rate.

Page size is also significant here. By having big pages, typically 4KB, we are unlikely to have TLB misses since the same page is accessed for this memory. Temporal locality will also help here, as the TLB will evict based on recency.

The TLB miss can be managed by either the CPU or the OS. Older **CISC** or complex-instruction set computers managed the TLB themselves, while modern **RISC** reduced computers raise an exception to be handled by the OS. In these computers, steps 4 and 5 are handled by the OS and step 6 only executes after the exception is handled.

The exception raised here is a little different from other exceptions. Typically, instructions that raise exceptions are skipped. Here, we want to retry the operation, so we need to get a different program counter.

The TLB cache is **fully associative** which means that any translation can be anywhere. This means that the VPN is encoded with the PFN and other bits.

One small note is that the TLB entries have valid bits just like the PTE, but they serve different purposes. In the TLB, it refers to a valid translation. If a context switch has occurred, for example, then all of the cache becomes invalidated. PTE valid bits refer to unallocated memory which results in a process kill.

Let's examine that cache invalidation. Flushing the cache on every context switch seems to miss the point of a TLB since it happens so often. What are other ways we can manage this? Hardware will typically add an **ASID** (address space identifier), which is similar to a PID but contains less information. This way, a TLB can contain information for multiple processes without accidental contamination.

## Smaller Tables

As discussed earlier, page tables can get real honking big, which is not good for memory consumption. TLBs can mitigate the performance problems of page table *access*, but we need better ways to mitigate memory usage of our page tables.

The obvious solution, also mentioned earlier, is to just increase the size of each page. The size of a page in \\(2^{bits}\\) is given by:

$$ addressSpace = numberOfPages + pageSize $$

so if we have a 32-bit address space, we can have 18 bits for our number of pages and 14 bits for our 4 KB page. 1 MB per page table is better because it's smaller, but there's a limit to this kind of strategy. If we make our pages too big, we will get **internal fragmentation** within each page, where an entire page is allocated but not that much memory within it is used, leading to waste. 4 KB is a good middle ground, which is why that's what x86 uses.

## Segmentation-Paging

We looked at segmentation earlier, but initially dismissed it due to its issues of variable size. What if we combined the two approaches into a hybrid? Instead of having one giant page table for the address space, what if we split it up into segments?

We can twist around the base/bounds registers to use them for a different purpose. The base register can hold the physical address of the actual page table for a particular segment, and the bounds will tell us where the physical end of the page table is. We can use these values to calculate the number of valid pages.

Again, let's use the top two bits to refer to segment. We'll figure out our base/bounds pair from the those bits, then get to our PTE by adding \\(VPN \cdot sizeof(PTE) \\) to it. The bounds register can be used to track our number of valid pages so they don't take up space in the page table if they are not used. However, this comes with the same issues of segmentation earlier: external fragmentation.

## Multi-level Page Tables

As is common for many problems, the solution for page tables is to put them inside another page table. **Multi-level page tables** are the de facto solution for page tables that are used in x86. However, this introduces a significant amount of complexity in our page table search, so we will need to examine that.

We will chop up our page table into its own kind of pages, each of which only contain PTEs. If an entire page of PTEs is all invalid, which means that none of them have valid translations, don't allocate that page. In order to manage this apparatus, we will introduce a new data structure: the **page directory**.

You can think of the page directory as a simple page table if the only memory being tracked was the sub-page table. Each entry has a valid bit which is true if *any* PTE in its mapped page is valid, as well as the PFN where the page of PTEs is stored. That PFN is only allocated if the valid bit is set.

If we organize this structure correctly, each chunk of the page table that we refer to as "pages" are actually page-sized and can fit into memory pages in kernel space. This greatly simplifies page table management.

However, there is a cost, as always. TLB misses require two memory loads in order to get the correct page because of the level of indirection that page directories introduce. Since TLB misses are rare, we take that tradeoff in return for significantly reduced memory consumption. This is a trade for space that sacrifices time.

Let's get some numbers for a multi-level page table. Assume a 14-bit virtual address space split into 8 bits for VPN and 6 bits for offset. Remember that this translates to 256 entries per table and a page size of 64 bytes.

We have to do some special bit magic to manage these page levels as well. In this example, our page table size is 10 bits; remember, it's VPN + PTE. We need to subtract our offset bits from that size to get 4 bits for number of pages, then subtract our PTE bits from that to get number of PTEs per page. In total, here is the formula:

$$ pageTableSize = numberOfPages + \underbrace{pageSize}_{PTESize + PTEPerPage} $$

\\(10 = 4 + (2 + 4)\\) in this example.

The page directory size is the same as the number of pages, so it is also 4 bits. When doing our translation, the highest order bits of the VPN are reserved for the page directory index. After indexing using these higher bits, the rest of the bits of the VPN are the "offset" within the page that the higher bits point to. You can think of this is as a mini address with the VPN being the PDI and the offset being the page table index (PTI).

## Infinity and Beyond

This example presented has only two levels, the page directory and the page table. But we can go deeper. Let's reset our numbers to 30 bit address space and 9 bit page size. The VPN is therefore 21 bits.

If we apply the same formula as before here, we end up with 7 bits for PTEPerPage. The lowest bits of the VPN will be reserved here. Our page directory will now have 16 bits (14 + 2 for each entry), but this is WAY too much. We need every piece of our structure to fit into a page, so we need to get the PDI to 7 bits or less.

The solution here is to further split the PDI into another VPN/offset split. The offset here will be 7 bits, as this is what fits into the page.

The zeroeth index is the VPN part, which is 7 bits. It can address 128 pages of the second-level directory. The second-level directory will address 128 pages of PTEs. This way, both the first and second level directory are 9 bits (index + entry size) so they fit into a page!

This is *really hard to understand!* Try to think of it where every level is a split between index and offset. The first offset must always be the size of a page, and every inner offset must be the size of page minus the PTE size. In this case, the first offset was 9 bits, and every directory offset was 7 bits. To get the maximum level \\(n\\), the formula is:

$$ addressSpace > (pageSize - PTESize) \cdot (n - 1) + pageSize $$

Our maximum level in this example is 3, because a level of 4 would result in \\((9 - 2) \cdot (4 - 1) + 9\\) which is 30, the same as the address space, which must be greater.

## Inverted Page Tables

A small coda to this discussion of paging is the **inverted page table**. Unlike other kinds of page tables, this is a page table that is shared between processes that maps *every physical page*. This massive table has information per entry about what process is using it, the virtual page number that the process using is referring to, and the physical page. PowerPC uses this model, with a hash table instead of an array to speed up lookups.

However, lookups are still slow, although they take up less memory. It is also difficult to implement sharing as specified earlier. You need to somehow chain multiple virtual addresses to one entry, which introduces serious complexity.
