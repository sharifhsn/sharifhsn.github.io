+++
title = "UDP Explained"

date = 2022-02-14



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

The simplest transport protocol that is used communicate between sockets is known as **UDP**, the **User Datagram Protocol**.

## UDP

Let's say we want to design the simplest transport protocol possible. All this protocol needs to do is get a message from an application at the socket and send it over the network to the other socket. However, there is one more function that it needs to perform: multiplexing/demultiplexing.

The *segment* (transport layer version of a datagram) must be directed to a specific socket. There's multiple sockets that applications communicate with and they all need to be multiplexed together to create a segment, which must then be demultiplexed for the end host to understand what socket it is sent to.

You can think of receiving multiple envelopes in your mailbox, then demultiplexing by reading who each envelope is addressed to and giving it to the member of your household to whom it is addressed. Then, when your household members want to send all of their mail out, you multiplex it into the mailbox.

UDP attaches a header with source and destination port numbers as fields for multiplexing, a length field, and a checksum to protect against packet corruption. Each of these fields is two bytes in length, resulting in a tiny 8-byte header.
