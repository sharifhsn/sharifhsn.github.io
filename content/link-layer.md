+++
title = "The Link Layer"

date = 2022-03-28



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

Through all the layers of the internet, the most essential is the **link layer**: converting bits to and from signals, detecting errors, flow control, and addressing.

<!-- more -->

## Encoding

We have spoken mostly through the guise of bits per second being sent across the internet and have left it at that. But what does it mean to send a bit over air waves? This isn't like a traditional electrical circuit or transistor in your computer; the internet uses signals.

The nuances of digital signal processing are beyond these notes, so we will simplify here. We can imagine signals like a pseudo-transistor where low signals are 0 and high signals are 1. However, this causes two issues. One is that time synchronization is impossible for waves that travel at the speed of light, and the other is that because signals are not emitting all the time, we need a way to differentiate between signals and simple noise.

The time synchronization is mandated by a clock standard such as the Manchester encoding, which is slower than the speed of light but allows each link to know exactly when a signal is coming.

## Framing

A **frame** is a group of bits in sequence. Frames are useful for manipulating data at the link layer, but they can escalate small bit errors to big problems. If a frame is too big, then it will error out too often, so we will often have smaller frames.

We also need a way to delineate the beginning and ending of frames. There are two ways, character stuffing and bit stuffing.

With character stuffing, a special meta character will be used to delineate frames. `^` is used for the beginning of frame (BOF) and `$` is used for EOF. If you need a `$` in your text, then simply add an escape character, like another `$` right beforehand.

With bit stuffing, a unique bit sequence `0x7E` or `0b01111110` will delineate frames. If that sequence appears in the data, it will insert a 0 within like `0b011111010`.

## Error Control

Inevitably, bits will be corrupted on a physical link. This is a reality of circuit engineering. We have discussed several ways to handle errors on the application and protocol layer; what about on the link layer? We can either request retransmission as with TCP, or we can make corrections to the errors automatically, which may or may not be possible.

Parity bits can ensure a fixed sequence so that the extra bit can protect against errors. For example, with even parity, there will always be an even number of 1s, so if a bit flip changes that, you know that there was an error. But multiple bit errors can escape this detection.

Another method is when the sender will send a checksum computed from the message divided into chunks along with the message so that the receiver can use the checksum, just like with other layers.

Polynomial codes utilize the high degree of uncertainty when computing `mod` of a polynomial. The remainder must be added to the message and it can be checked by the receiver. This is a more complex form of error detection mathematically.

## Addressing

We have heretofore defined a host as an IP address. But how do we identifies hosts at the link layer? For links we use a **MAC address**, which is a unique 48 bit address to a device which will communicate through the internet. MAC addresses are far less sophisticated than IP addresses. They cannot be grouped or categorized like with NAT. The MAC address of the destination is included in the frame that is sent. They are also unique to types of links, so a device will have different MAC addresses for Wi-Fi, Bluetooth, etc.

Hosts can then interact in multiple ways. One way is *access point*. Hosts must communicate with an access point which acts as an intermediary between hosts. Another is *ad hoc*. When you set up your Amazon Echo for the first time, you connect with it ad hoc via Bluetooth on your app and set it up from there to program it. 

## ARP

When connecting via LAN, hosts are part of the same subnet. In order to translate the IP address to a MAC address, the hosts use a protocol called **ARP (Address Resolution Protocol**. This is a simple protocol which defines its own identification and a source/destination address, as well as a broadcast `FF` packet. We know the sender IP, MAC, and the destination IP, but we don't know the destination MAC address, so that field is blank. The ARP packet will traverse the entire link, and if a host recognizes its own IP address, then it will send an ARP reply back to the source with its MAC address.

## LAN Extension

LANs seem pretty nice. Why don't we just use LANs everywhere? Bandwidth is limited, so you need to limit the amount of hosts that can connect on one network.

A *learning bridge* is a type of access point that connects LAN segments. It maintains a table of hosts that are connected to the network. It intermediates between the hosts, receiving and sending packets as necessary if packets are sent between LANs. For each packet, the bridge stores the MAC and port of the packet and then floods all the LANs if it can't find a match.

## Multiple Access

If two packets are sent at the same time, that is called *collision* or *interference*. We can only have one pair communicating at a time. We must have a multiple access protocol to make sure that only one access method is used at a time. You can think of being in a class; if everyone spoke at once, it would be mayhem! There must be an extra overhead of each person raising their hand in order to ensure correct communication takes place.

## Wireless

When a host sends a wireless signal that bounces off of a satellite to a receiver, how do we know that interference has occurred? We could use a TCP packet timeout, but that takes a while. A neat trick that you can do is leverage the fact that satellites reflect all signals to everyone, including your own. A host can receive its own signal and check if it was interfered with.

Originally, **pure ALOHA** was used, based on the needs of a university in Hawaii. This uses the broadcast method. If it receives garbage back, it will sleep for a random amount of time and retransmit. The reason it is random is because if both the hosts that sent garbage data send the packets again at the same time, it will cause *another* collision! For transmission time \\(t\\), the vulnerable period where packets will be destroyed is \\(2t\\).

**Slotted ALOHA** has better checks to stop collisions from occurring in the first place by setting slots where hosts will send packets, which means that they wouldn't collide in different slots.

But the best throughput is done if we check the actual channel before transmitting the packet, so we don't have to guess or check after the fact if a collision happened. For this we need **CSMA (Carrier Sense Multiple Access)**. If the channel is sensed and there are packets travelling, the host will wait to send the packet then immediately send a packet when free. This is a persistent system. The problem with this is that this will cause a collision when multiple hosts are waiting for a channel. We can solve this the way we solved ALOHA. If each host sleeps a random amount of time, even a small amount of time, it will give the hosts time to sense the channel.

## Contention Access Methods

In order to sense the channel, there are many methods.

**1-Persistent CSMA** will constantly bug the channel to check if the channel is idle. If it's busy, transmit and check again. This only really works when there is one host trying to connect.

**Non-Persistent CSMA** will transmit, then sleep for a random amount of time before trying again if the channel is busy. This means that that collisions won't happen. This is better than the previous method, but can be slow.

**\\(P\\)-Persistent CSMA** has the same idea, but instead of being random it will transmit again with probability \\(P\\). This fine-tunes the previous method for our specific needs and improves utilization.

Ethernet uses the **Ethernet Backoff Algorithm**, which utilizes binary exponential backoff. If a collision happens, then it will pick a time slot to transmit again out of \\(2^k\\) slots, where \\(k\\) is the number of collisions that have occurred. The length of each time slot is equivalent to. For example, let's say two hosts collide on an Ethernet connection. Both hosts will both, on their own, divide the next few seconds into two slots and randomly pick between them. If they each end up in different slots, they'll transmit fine. If they end up in the same slot again, repeat the process but with four slots. As you can see, the chance of collision will dramatically decrease with more slots. After a maximum of sixteen slots, if there are still collisions, then your host will give up and say that the link is down.

## Ethernet

Ethernet is a wired multiple access protocol defined by IEEE 802.3. As previously stated, it uses 1-persistent CSMA with the backoff algorithm.

Ethernet frames have a preamble full of special information, as well as the source/destination MAC addresses already mentioned. The header will also say the type of protocol being used, whether it is IPv4, IPv6, ARP, RARP, etc. Finally, at the end, a checksum will be stored so that we can check if corruption has occurred.

| Preamble | Source MAC | Dest. MAC | Type | Data | Checksum |
| -------- | ---------- | --------- | ---- | ---- | -------- |

## Multiple Access Channel Partitioning

We have previously assumed that all hosts are fighting for the use of one channel and will have to retry connections if collisions occur. One way to solve this is to have predetermined allocation of channel access. This way, there is no chance of collision because each user will wait its turn on its own. However, this can introduce wastage if users are assigned to a channel partition but end up being idle. The division is governed by different techniques.

**TDMA (Time Division Multiple Access)** will divide the spectrum across time. Like how OS schedulers will give processes exclusive access to the CPU for a certain amount of time, so too will users be granted exclusive access to the channel for a certain amount of time.

**FDMA (Frequency Division Multiple Access)** will divide the spectrum across frequencies. The most common example of this is for radio. Radio channels are exclusive to certain frequencies.

**CDMA (Code Division Multiple Access)** will send signals in a coded format. Imagine a large group of people that are speaking all at the same time. If everyone is speaking in English, the message will quickly get garbled. However, if every conversation has its own language, then the message will be clear. Even if I hear a conversation next to me in Chinese, I will tune it out because I don't know Chinese and be able to speak and hear in English. There must be some amount of power control so that one person does not speak too loud, but otherwise the communication will work.

## Wi-Fi

Computers can be connected through a sort of wireless LAN. If connecting a computer that does not have a screen or an otherwise easily accessible user interface, it is often convenient to connect with it using ad-hoc mode to a device with a screen so it can be configured, like Amazon Echo.

Each local set of computers that can connect in these ways is known as a **BSS (Basic Set Size)**
