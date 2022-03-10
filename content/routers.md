+++

title = "Routers"

date = 2022-03-10



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

Everyone nowadays gets the Internet through the **router** in their home. But what *is* a router?

<!-- more -->

## Speed

It's likely that you have an internet speed in the tens of Mbps. This unit is **megabits per second**, or one million bits per second. Some routers in important locations have speeds in the Gbps, which is 1 *billion* bits per second. This is incredibly quick. How do routers accomplish this?

## Structure

The router is composed of a **control plane** and a **data path**. The control plane contains all the necessary external tooling to determine the paths. It contains routing protocols that need to be implemented, as well as a routing table. This table is used to look up all the possible paths to the destination and pick the shortest one.

This information is all stored on the data path, where each packet is actually processed. The routing table shortest path is put into the **forwarding table** which typically has a very small amount of SRAM in order to quickly mediate routing. You can think of it almost like a register that is close the process as opposed to the main memory where the calculations are made.

> Remember this distinction! The *routing table* contains all possible routes from this router, kind of like Google Maps. It needs to be a big table, so it is usually stored in slow, inexpensive DRAM. The *forwarding table* only contains the best possible route, so it is a small table stored in fast, expensive SRAM.

Routers must also have input and output buffers. The input buffer is needed because the router may take in packets faster than it can process them, so it needs a place to put them before it can figure them out. The output buffer is needed because of the opposite problem; if the router processes the packets faster than the output link can output them, the packets need a place to be.

## Forwarding Engine

Let's examine the forwarding table in a little more detail. The forwarding table is a simple table that looks like this:

| Dest-network  | Port |
| ------------- | ---- |
| 65.0.0.0/8    | 3    |
| 128.9.0.0/16  | 1    |
| 149.12.0.0/19 | 7    |

It is a simple path from networks, remembering which port to send packets.

You might have noticed that our destination networks are in prefix form. The one on port 7 has the longest prefix because it is matching the first 24 bits. We must match the longest prefix form first when looking for the network. These prefixes are the most specific and therefore must be matched before the smaller, more general prefixes. Prefixes can overlap, so we must pick the more specific one. It is typically not explicitly linear, a hash table-like structure is used to make lookups faster.

In fact, **hierarchical addressing** is important for allowing us to aggregate routes. Different organizations within the same ISP network will have some bits after the network bits devoted to differentiating between organizations. Let's imagine that Rutgers Newark and NJIT are on the same ISP network, but are different organizations, obviously. The first 19 bits of that prefix might be devoted to the ISP network, then the next 4 bits will be used for organizations within that network.

## Switching Fabrics

Let's consider each port as form of input and output. We will need a way to switch between devices. There are multiple ways to implement **switching fabrics**.

One way is through **memory**. This is the cheapest way of going about it. The I/O device will take the network and write it into memory. We can use DMA (Direct Memory Access) to do this more quickly. The header is processed, and the router checks the routing table for the best route. The packet must be copied from the port into memory, then processed, then loaded back into the output port. This is very slow. Most of the bottleneck is in the memory bus when copying memory. These routers are typically less than 10 Mbps.

Another way is through **bus**. The packet is transferred between input and output through a I/O bus, with a memory buffer to hold the packet on each end. There's no CPU movement here, it's all I/O instructions. But we still need some kind of command for the destination port number. The CPU will periodically calculate and *cache* the forwarding table on the input device (also known as **line card**). If there is a cache miss, then I/O will ask the CPU to update the cache from main memory, which is slow but doesn't happen often. However, since there's only one bus, there is only one communication possible between one port and another at a time. This speed is in the high Mbps.

The third way, which is the most high-end way, is the **crossbar**. It works like a bus, except that there exists a bus between every single port, which means that for \\(n\\) ports, we can have \\(\frac{n}{2}\\) simultanenous transfers. This is in the best-case scenario where each input goes to a different output. However, if two ports are trying to connect to the same port, then one of them will be bottlenecked because it's still one bus. However, this complicated switching fabric is extremely expensive, which scales with the number of ports. The price can be worth it for a company like Google; their speeds can be in the Tbps!


