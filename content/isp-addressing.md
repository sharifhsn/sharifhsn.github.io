+++

title = "ISP Addressing"

date = 2022-02-28



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

How does an ISP assign IP addresses to its customers?

<!-- more -->

An ISP has a block of addresses that are partitioned to its customers. If an ISP network has an address like `200.8.4/24` address, that is 256 addresses. `/20` is for 4K hosts, and `/16` is for 64K hosts. In fact, the calculation is \\( 2^{32 - n} \\).

## Subnetting

A network can be subdivided into **subnets**. This way you can have each router handling a smaller portion of the network, or have different kinds of routers i.e. wired/wireless handling different subnets.

In order to divide IP addresses, we use a **subnet mask**. For example, if our network is `128.64.32/24`, a range we could have is `128.64.32.0-127` and `128-255`. The easiest way to mask this is to look at the most significant bit. In the first range, the most significant bit is 0, and in the second range, the most significant bit is 1. The mask in this case would be `255.255.255.128` or in hex, `F.F.F.8`. If we bitwise AND this mask with the IP address, we will get the correct subnet.

We can follow the same procedure to get further divisions by the power of 2. We can mask over two bits to get four subnets of ranges `0-63`, `64-127`, `128-191`, and `192-255` with a mask of `255.255.255.192`.

Let's think of an analogy. If we want to deliver some mail to our neighbor, the easiest way to do it is to go directly to his mailbox and give it to him instead of passing it off to the post office. In the same way, we can quickly send messages between hosts on the same subnet.

In order to facilitate this, the router will first check if the sender and destination (both first &ed with the subnet mask) are in the same subnet. If they are, it doesn't bother sending the message over the Internet and it will actually just directly send the message to the destination itself.

In particular, this makes email between people on the same networks extremely quick. This is why it's nice to have email between people with the `scarletmail.rutgers.edu` domain.

## IPv4 Header

In order to get a packet to a destination host, we need a packet header with the identity of both the destination identity and the source identity. At this layer, we don't worry about reliable exchange, dropping packets, sequence numbers, etc. That's for TCP to handle, our only job is to send the packet to the destination.

However, we have to detect some kinds of problems. We don't want to loop packets over and over, this will overload the network. There used to be a worry of fragmentation, if the size of the packet is greater than the *maximum transmission unit* (MTU) of the router. Nowadays, everyone has broadband so this is not an issue because everybody has high link speeds. When IPv4 was invented we needed checksums for verification if there are bit flips. IPv6 addresses have much less checksum machinery because it's a waste nowadays, it only checks at the end. IPv4 has to calculate a checksum at every router, which catches errors early but is generally wasteful.

The *time to live* (TTL) is 1 byte long and is used to protect against loops. It starts at 255 and decrements every time it passes through a router. If the TTL is 0,  the packet is dropped and a "time exceeded" error is thrown to sender. This prevents the packet from being looped around forever.

An entire 4 bytes is devoted to fragmentation machinery. This is not in IPv6, which simply throw an error if the packet is too large. The reason this is fine because it is a rare case nowadays and it cuts fat out of the header. Here are the steps of fragmentation:

1. router receives packet larger than MTU

2. if *Don't Fragment* (DF) flag is set, throw *Fragmentation Needed* error

3. else, divide the packet into fragments maximum size `outgoing MTU` - header size (20 bytes)

4. in each new packet fragment, the `length` field is the size of the fragment, the *More Fragments* (MF) flag is set except for the last fragment, and the `fragment offset` field is set to the offset of the fragment in 8-byte blocks. The checksum is recomputed for the fragment. [IPv4 - Wikipedia](https://en.wikipedia.org/wiki/IPv4#Fragmentation_and_reassembly)

## ICMP

What do we do to propagate errors? We're already using IP for sending messages, how do we send errors? This what the **ICMP** protcol is for. It is unreliable like UDP, and is tightly coupled with the implementation of IP. Its protcol ID is 1, which signifies its importance.

There are certain known error codes. The *echo request/reply* also known as *ping* is simply a request to check if the host is alive and responding. There is also the **traceroute**, which records the route that a packet takes by tracking TTL. As we know, when a router receives a packet, it decrements TTL. If we start TTL at 0 and slowly increase it, it will throw time exceeded back to us from each router in order. This way, we can see every router on the way to our destination.
