+++

title = "More Internet Protocols"

date = 2022-03-03



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

Besides the primary Internet protocols, there are some others to learn about, like **DHCP**, **NAT**, **IPv6**, etc.

<!-- more -->

## DHCP

What happens if a device doesn't have a permanent IP address? You take your phone around multiple mobile networks, there isn't a consistent IP address between them. How do you access the internet without an IP address?

**Dynamic Host Configuration Protocol** is a client-server protocol that is used in these scenarios. As the name implies, it allows for dynamic IP address allocation that are leased for a certain amount of time. Configuring things like the subnet mask, gateway configuration, etc. to set up an IP is complicated and not feasible for the average user. Imagine if you had to do all that setup everytime your phone moved to a new location!

DHCP has two main components: the protocol for delivering the bootstrapping information from the server to clients, and the algorithm for dynamically assigning addresses to new clients. How do you start a connection from nothing?

In order to allocate a new address, there are three modes. Automatic allocation gives permanent address, dynamic allocation leases addresses, and manual allocation is managed by a system administrator.

DHCP sends packets over a socket at port 67. Its IPv4 header indicates that it is protocol 17. The UDP packet is sent with no initial IP address because there is none. It is sent with the broadcast server IP address `255.255.255.255`. The protocol must also interface with the link layer at the client's MAC address; remember, we're starting at zero. There is also space for the server hostname, boot filename, and several kinds of options.

The options indicate what the purpose of the UDP packet is. For example, option 1 is `DHCPDISCOVER`, which is the message that lets the server know that a client is looking for an IP address. The communication sequence is directly encoded into the options in the packet. Some others:

- DHCP Offer: server response with parameter proposal

- DHCP Request: like discover, but focused to a specific server

- DHCP ACK: server gives IP address to client

- DHCP NAK: server declines to give IP address to client

- DHCP Decline: client declines the given IP address

- DHCP Release: client gives up its IP address

DHCP typically applies within a subnet. Relay agents on routers, like with BOOTP, allow servers to handle requests from other subnets.

## NAT

There are two kinds of IP addresses: *public* and *private*. Private addresses are reserved for `10.0.0.0` to `10.255.255.255`. If you send a request to private address, your router will not send it out to the Internet. Private addresses are also free; you can hand out as many as you want without it costing anything.

However, requests sent from a private IP address cannot access the Internet. In order to access the Internet, we use a **Network Address Translation** box. This NAT box is assigned a single public IP address and it is the public-facing IP for all of the machines with private addresses on it.

This is likely how your home router works. Each device in your home only has a private address and every time it sends a request to the Internet, that request is sent to the NAT box in your router and translated to one single public IP for all of the machines in your house.

The NAT box will take in packets that are sent from private IP addresses and record the packet address in a table, then replace the IP address with its own and its port with some random port. When packets are sent back to that address, the NAT box consults its table and sends the packets back to the correct private IP address based on the port it was sent on.

Although there are technically \\( 2 ^ {24} \\) addresses, in practice you can only have less than \\( 2 ^ {16} \\) addresses because each of those needs to have an associated port, and that's a 16-bit number (minus some reserved ports).

But why would you do this? Remember that IPv4 is a 32-bit number for addresses, which only allow for \~ 4 billion unique devices. Clearly, there will be, and perhaps already are, more devices that use the Internet than that. IPv6 was created in part to solve that problem, but as many devices only accept IPv4, this works in the meantime. It also provides security to users since ports aren't accessible from the public, they are instead randomly assigned. Even if someone's IP address is tracked, the malicious agent still doesn't necessarily know what device the request is from.

## IPv6

IPv6 is an updated protocol from IPv4 that is optimized for the needs of the modern day. It is significantly simplified compared to IPv4 as much of the machinery in an IPv4 header relates to problems that generally no longer exist. However, the header size is also 40 bytes, twice as large as the IPv4 header. This is because the new source and destination IP address that must be stored in the header are now 16 bytes, not 4, the vast majority of the header size is dominated by address size.

IPv6 does not protect against some problems that still exists, such as bit flips. If that happens, the packet is simply dropped because a different layer will do a checksum.

| Version        | Traffic Class  | Traffic Class  | Flow Label     | Flow Label  | Flow Label  | Flow Label | Flow Label |
| -------------- | -------------- | -------------- | -------------- | ----------- | ----------- | ---------- | ---------- |
| Payload Length | Payload Length | Payload Length | Payload Length | Next Header | Next Header | Hop Limit  | Hop Limit  |

followed by the source address and destination address, along 4-bit boundaries
