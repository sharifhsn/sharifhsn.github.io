+++
title = "Packet Switching vs Message Switching"
date = 2022-01-31

[taxonomies]
categories = ["Internet Technology"]
tags = ["cs352"]
+++
Propagation de

## Performance

**Performance is bottlenecked by propagation delay and transmission time.**

Propagation delay is dictated by the physical distance between the client and server. If you were to send a message to server on Mars, there would be 200 million km to travel across, aka 1000s at the speed of light! New York to Los Angeles is about 20ms at this same speed of 5μs/km.

Transmission time is dictated by bandwidth, which tells you how many bytes per second you can send. 1MB/s link speed will take 1ms for 1000B. The reason this happens is because as soon as the first bit is propagated, all the rest of the bits will follow which will be concentrated by bandwidth, 1μs after each other with this link speed.

For small messages in the tens or hundreds of bytes like handshake acknowledgements, propagation delay is very significant. For larger messages, propagation delay is ignored and bandwidth is considered more significant.

$RTT = 2PD$ : Round-trip time is equal to twice the propagation delay.

### Message Switching vs. Packet Switching

**Packet switching is faster because it is a continuous stream.**

Assume 1000B/s link speed between sender and destination. If we do message switching, it is slow. A 1000B would take 1s (duh) to go from sender to router. Similarly, it would take another second to go from router to destination, so 2 seconds in total.

Let's assume we split our 1000B message into 100B packets. The first packet will take 0.1s to arrive to the router and another 0.1s to arrive at the destination, so 0.2s. Each packet follows directly after the other, so the second packet takes only 0.1s total because it's already at the router by the time the first packet is at the destination. In total, the message takes 1.1s to travel.

These benefits compound when messages are passing over multiple routers, because only the first packet has to deal with the overhead of the link speed for all the routers. All the rest of the packets will only take the time for the link speed between the last router and the destination.

However, bottlenecks can still occur when link speeds between routers are different. The weakest link will bottleneck the connection even in packet switching.
