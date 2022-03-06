+++
title = "TCP Explained"
date = 2022-02-20

[taxonomies]
categories = ["Internet Technology"]
tags = ["cs352"]
+++
TCP is the well-known protocol for reliable transfer of data. But how does it actually work?

<!-- more -->

**System of handshakes ensures reliable transmission of data.**

- What happens if a packet is corrupted, and its bits are flipped?
  
  - Use a checksum for each packet.

- What happens if a packet is lost somewhere?
  
  - Wait for acknowledgement from receiver, and if not received, resend.

- What happens if a packet is duplicated? etc. etc.

There are lots of problems that can happen with UDP. Let's create a protocol that doesn't have this unreliability.

Our new protocol has a sequence number for each packet, either 0 or 1. This is known as an *alternating bit protocol*. When it sends a packet, it waits for ACK (acknowledgment) before it sends the next packet. This will take one RTT (round trip transmission) per packet. This is obviously pretty inefficient, but it does solve at least some of these issues. It solves the duplicate issue because if it sends the same packet, it will have the same bit sequence number and hterefore will not be taken into account. You can think of this as an infinite loop which flips its break condition after being broken. This is possible because you're only sending one packet at a time. Here is time and utilization.

$$ T_{transmit} = \frac{L_{packetLength}}{R_{transmissionRate}} $$

$$ U_{sender} = \frac{\frac{L}{R}}{RTT - \frac{L}{R}} $$

If we can send W packets at aa time, then we can replace $\frac{L}{R}$ with \\(W\\), which will improve our time by a factor of \\(W\\)! However, if we send a stream of packets, then there are more issues. If you send a receiver too much data, then it will throw out the data it cannot receive. If packets get lost somewhere in the router, you will have no idea. Stopping the stream to check for issues wastes a lot of time.

We can solve this using a window time. We send packets up until the first ACK is received, then we check for any problems with ACK. But what is the window size?

$$ B \cdot RTT = W \cdot packetSize $$

This way we can pipeline as many packets as are possible before anything can go wrong. This increases the complexity on the recv side, but keeps the connection simple and reliable. Let's say the window size is 3. We can send 3 packets, then we must wait for ACK for the first packet. When the ACK is received for packet p, we send packet p + 3. If the timer expires, we have to resend the packet. We can make the assumption that everything to the left of the window has been ACKed, and everything to the right of the window has not yet been sent.

What is the efficiency of this new method?

$$ U_{sender} = \frac{3 \cdot \frac{L}{R}}{RTT \cdot \frac{L}{R}} $$

This is an increase of a factor of 3!

But what if a packet is dropped and sent later? This is where the receiver comes in. There are two ways of dealing with this:

1. Go back N: simple, keep a buffer of 1 and throw out data with greater sequence numbers and ACK at that packet that was received. This will force the sender to resend all of the packets thata re now timed out. However, this also throws out correctly received packets.

2. Selective Repeat: complex, keep a buffer proportional to W and keep data that has been received, cancelling the timers for each. Once the data has been correctly received, you can move the window and keep the buffer open for anything that hasn't been received. Let's say packet 1 is timed out but 2 and 3 came in correctly. We can cancel those two timers and slide the window to 4 and 5 but still wait for 1. *There are only ever W packets in transmission at any given point!*

In order to demonstrate the benefits of selective repeat over go back N, we'll work though an example. Say we are sending 5 packets of size 100 B each over a 100 MB/s link with a RTT of 100 ms, with packet timeout being send_time + RTT and the window buffer being of size 5. How long would it take to receive all of the packets if we drop the third packet?

The send_time here is 100 B / 100 MB/s or 0.001 ms. The time to receive a packet is half of RTT or 50 ms. The timeout length is 100.001 ms.

With go back N, packets 1 and 2 are received at times 50.001 ms and 50.002 ms. All good! But when the third packet is dropped, we must wait 100.01 ms to receive it again at 150.003 ms. Even though we received packets 4 and 5 already at 50.004 ms and 50.005 ms, we had to throw them out because we didn't get packet 3. We resend them as soon as possible after packet 3 so they are received at 150.004 ms and 150.005 ms respectively. The whole transaction takes 150.005 ms.

With selective repeat, the four non-dropped packets are received in the same way. However, after packet 3 times out and is resent, it is the only packet that is sent at 150.003 ms, since every other packet was within our window. The whole transaction here will take 150.003 ms.

The protocol we have just created is known as **TCP**.

- connection management

- retransmission

- flow control

- congestion control

- frame format

It has it all!

TCP is the default protocol for every web browser in existence and is the de facto protocol for reliable transmission. Chrome now also uses QUIC which is a Google-created protocol but the vast majority of Internet traffic is still TCP.

## TCP Header

The TCP header is much more complex than the UDP header. It includes some of the same elements, such as the port numbers and the checksum, and others:

- 32-bit sequence number and acknowledgement number, which are used for reliability

- 16-bit receive window for *flow control* so that the bytes sent do not overwhelm the receiver

- 4-bit header length field to indicate the length of the *header*, needed because of variable options, but usually 20

- variable options field needed when specific maximum segment size (MSS) is needed or other options for high-speed networks

- 6-bit flag field: ACK for a segment that is an acknowledgement; RST, SYN, and FIN for connections, PSH for immediate network sending, and URG to indicate some urgent data is located at the location where the 16-bit urgent data pointer points (these last three are typically unused)

The sequence and acknowledgment numbers are how TCP messages are ordered despite being sent in a full-duplex way. The sequence number orders the segments for reconstruction into a full message, and the acknowledgment number is a way of communicating the next byte that the sender needs. This way, if a segment is dropped, the sender of the dropped segment knows that the receiver didn't get it because they're still waiting on it in the acknowledgement number.

## RTT

Round-trip time is an important concept when we consider making our protocol as fast as possible. This is how long it takes for a segment to be sent and an acknowledgment for that segment to be recieved. In order to get an accurate measure of the RTT, TCP takes the current RTT and refines it to an average as more segments are transmitted, since any one SampleRTT (SRTT) might be an outlier. In order to get the average EstimatedRTT we follow this formula:

$$ERTT = (1 - α) \cdot ERTT + α \cdot SRTT$$

\\(α\\) in this formula is typically 0.125. This means that average is weighted towards recent samples, since they better reflect the current weight.

We should also have a measure of variance in RTT, since averages can be deceiving. DevRTT is the measure of the variance, which is calculated like this:

$$DevRTT = (1 - β) \cdot DevRTT + β \cdot | SampleRTT - EstimatedRTT |$$

Like ERTT, DRTT moves and is weighted recently, with a \\(β\\) value of usually 0.25.

The reason this is important is because the interval at which a TCP segment transmission times out is based on RTT. TCP calculates this way:

$$TimeoutInterval = ERTT + 4 \cdot DRTT$$

This starts out as one second, then changes over time. Timeouts cause this value to double because a new link may be chosen that has a longer RTT than what ERTT has estimated so far.

## Congestion Control

A network is said to be **congested** when so much traffic is being sent through the network that many packets become lost. To combat this, TCP implements **congestion control**.

Every sender has a limited rate at which it can send traffic into its connection based on the way TCP thinks the network is congested. If it thinks it's not very congested, then it increases the send rate, and vice versa. There are three elements to this implementation: how to limit the rate, how to perceive the congestion, and how mathematically the rate should be limited.

The mechanism at the sender keeps track of a *congestion window*, also known as `cwnd`. The amount of unacknowledged data that a sender sends must not be greater than either the congestion window or the receive window.
