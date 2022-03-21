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

The centralized node is the one that does all the calculations for shortest path. This is known as a **distance vector**. Each router must calculate its own distance on top of the other links to the node that it must pass by. Think of a table that is passed from link to link which is updated according to Dijkstra's algorithm. Degenerate longer paths are thrown out when tables "merge" at a router.

If a link breaks, then the routers that have broken links get reset in the final table. When the broken router sends a new distance vector table, it updates the central table in the same process as the table generation.


