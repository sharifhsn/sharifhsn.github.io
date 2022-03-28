+++
title = "Routing Algorithms"

date = 2022-03-21



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

When a host needs to find the best destination/prefix to another host, what **routing algorithm** does it use? We can use either *intra* or *inter* domain.

<!-- more -->

## Routing and Forwarding

As you may recall, the *routing table* is a large DRAM table containing all possible paths and the *forwarding table* is a small SRAM table containing only the best possible path. The router also caches previous best paths so that they don't need to be retrieved for the same host over and over from DRAM.

You can visualize multiple routers in an undirected weighted graph. The weights matter so that a slow link will not be chosen even if the path is technically slower. By maintaining all paths, the router can pick a new best path if a link in the route crashes.

But how does the router actually update the forwarding table? There are several different algorithms we can use.

## Link State

**Link state** is an intra-domain method that works fine for 50-100 routers, say on a university campus.

Let's use the graph analogy again. It is often useful to abstract concepts in networking to graphs when thinking about algorithms, since we can apply well-known concepts from mathematics and algorithm design to the problem. There are multiple factors that play into the weight of each link, not just link speed, but routers assign their own holistic score to links for simplicity.

The obvious choice here is to use [Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) for shortest path, as it applies to undirected weighted graphs like the on we have just constructed. This algorithm generates the shortest path to every path up to and including the destination. It requires knowing all of the link speeds beforehand. This is accomplished through a *link state broadcast* i.e. pinging every router and checking the link speed of the ping. The algorithm is also iterative, which means the number of shortest destinations scales with iterations linearly. The time complexity of the algorithm is \\(Θ(|E| + |V| \log |V|)\\).

This algorithm works best in a centralized graph like at a university. Instead of sending a link state broadcast every time a connection is made, each node in a graph will send its own **link state** to a centralized location, and that location will be checked for recomputation of shortest path.

The centralized node is the one that does all the calculations for shortest path. This is known as a **distance vector**. Each router must calculate its own distance on top of the other links to the node that it must pass by. Think of a table that is passed from link to link which is updated according to Dijkstra's algorithm. The initial distance vector table for a router only contains the link speeds for its adjacent routers, and all others are marked as infinity. Degenerate longer paths are thrown out when tables "merge" at a router.

If a link breaks, then the routers that have broken links get reset in the final table. When the broken router sends a new distance vector table, it updates the central table in the same process as the table generation.

However, this doesn't work well when we have different domains in different autonomous systems controlled by different ISPs. We need some kind of inter-AS algorithm to manage this. However, as the intra-AS algorithms works well, we use *federation* in our algorithms. Inter-AS algorithms are used to route between ASes and intra-AS algorithms are then used within the AS.

## BGP

There are some barriers to inter-AS routing. We might not know whether the host is reachable, so we need to ask nearby ASes if they know. The protocol that these ASes use is **BGP**, the **Border Gateway Protocol**. With BGP, ASes can

- obtain subnet reachability information from neighboring ASes

- propagate reachability information to all the routers within the AS

- use this information to calculate the best route to another subnet

BGP works through persistent sessions between BGP routers over semi-permanent TCP connections. The sessions can be through any router, there doesn't need to be a physical link. The two peers can communicate regardless of the physical router.  This is the sequence of steps:

1. Establish TCP over port 179.

2. Exchange routing tables. At first, this is a lot of data, but after a connection is established only deltas (like Git) need to be exchanged.

3. Send four kinds of messages.
- *Open* - establish session by exchanging AS numbers and the BGP identifier (arbitrary router IP address). There is a timer for how long to wait before just assuming the session is down i.e. in an outage.

- *Notification* - report unusual conditions, usually an error. If there are header errors, or a timer has expired, etc. then the TCP session must be terminated.

- *Update* - inform if there are new active or old inactive routers. The message includes the withdrawn routes, which are inactive routers, the length of that field as it can be variable, etc.

- *Keepalive* - regular message that the session is alive. BGP needs this regular message otherwise the neighbor AS won't know whether you're alive or not.

When we say that a router is advertising a prefix, there are some necessary assumptions. The routing information must be valid, and does not need to be refreshed until necessary. The path that an AS advertises to a node is the same path that AS uses to communicate with that node.

The BGP protocol additionally has many attributes that indicate the characteristics of a prefix:

1. ORIGIN: advertises the origin of an announcement and prefix injection. This is manually configured by intra-routing protocols.

2. AS_PATH: a link of ASes which is the path of ASes that the prefix has traveled to. This is useful for detecting loops and selecting shorter routes of ASes.

3. NEXT_HOP: the hop field is when you cross the AS boundary. After you cross, the next hop value is the next AS you need to get to in order to get to the destination router. The reason this is important is because the IP packets within an AS can travel in any order irrespective of BGP, so the intra-routing algorithm can decide where to go based on this value. This becomes significant when a network experiences *transit traffic* and different networks are using that network as an intermediary. In order to get the packet to exit the network as quickly as possible, BGP can make the decision to send the packet through non-BGP routers if it's a faster way to make the packet leave the network.

4. MED: stands for Multi-Exit Discriminator. If ASes are connected via multiple links, the AS that receives a prefix uses this value to discriminate between exits, with a lower value being better.

5. LOCAL_PREF: indicates a local preference for a specific prefix, is a number that is more for more preference, by default 100. For example, if outbound traffic is preferred to be a specific exit point, then that point will have high local preference. It can be used to break a tie between routers.


