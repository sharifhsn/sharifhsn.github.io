+++
title = "TCP Explained"
date = 2022-02-26

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

$$ T_{transmit} = \frac{L_{packetLength}}{R_{transmissionRate}} $$$$ U_{sender} = \frac{\frac{L}{R}}{RTT - \frac{L}{R}} $$

If we can send W packets at aa time, then we can replace \\( \frac{L}{R} \\) with \\(W\\), which will improve our time by a factor of \\(W\\)! However, if we send a stream of packets, then there are more issues. If you send a receiver too much data, then it will throw out the data it cannot receive. If packets get lost somewhere in the router, you will have no idea. Stopping the stream to check for issues wastes a lot of time.

We can solve this using a window time. We send packets up until the first ACK is received, then we check for any problems with ACK. But what is the window size?

$$ B \cdot RTT = W \cdot packetSize $$

This way we can pipeline as many packets as are possible before anything can go wrong. This increases the complexity on the recv side, but keeps the connection simple and reliable. Let's say the window size is 3. We can send 3 packets, then we must wait for ACK for the first packet. When the ACK is received for packet p, we send packet p + 3. If the timer expires, we have to resend the packet. We can make the assumption that everything to the left of the window has been ACKed, and everything to the right of the window has not yet been sent.

What is the efficiency of this new method?

$$ U_{sender} = \frac{3 \cdot \frac{L}{R}}{RTT \cdot \frac{L}{R}} $$

This is an increase of a factor of 3!

But what if a packet is dropped and sent later? This is where the receiver comes in. There are two ways of dealing with this:

1. Go back N: simple, keep a buffer of 1 and throw out data with greater sequence numbers and ACK at that packet that was received. This will force the sender to resend all of the packets thata re now timed out. However, this also throws out correctly received packets.

2. Selective Repeat: complex, keep a buffer proportional to W and keep data that has been received, cancelling the timers for each. Once the data has been correctly received, you can move the window and keep the buffer open for anything that hasn't been received. Let's say packet 1 is timed out but 2 and 3 came in correctly. We can cancel those two timers and slide the window to 4 and 5 but still wait for 1. *There are only ever W packets in transmission at any given point!*

The protocol we have just created is known as **TCP**.

- connection management

- retransmission

- flow control

- congestion control

- frame format

It has it all!

TCP is the default protocol for every web browser in existence and is the de facto protocol for reliable transmission. Chrome now also uses QUIC which is a Google-created protocol but the vast majority of Internet traffic is still TCP.
