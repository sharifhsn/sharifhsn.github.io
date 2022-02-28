+++
title = "DNS Explained"
date = 2022-01-31

[taxonomies]
categories = ["Internet Technology"]
tags = ["cs352"]
+++

## Protocols

**We need protocols to define the way that messages are interpreted.**

Messages are different based on what kind of information you are sending. For example, *HTTP (HyperText Transport Protocol)* defines the way that webpages are served. Otherwise, the bytes that are sent between a client and a server are meaningless.

## Identification

**We can identify hosts using IP addresses and ports.**

With cell phones, we use telephone numbers to identify each other and call each other. Sometimes, we can store names in contacts associated with the phone numbers that are easier to remember than unique numbers.

*IP addresses* work the same way. Each IP address uniquely identifies a host then can send and receive information through a network. IPv4 is a 32-bit number that was traditionally used, however due to its limited scope (only $2^{32}$ ≈ 4 billion possible IP addresses). IPv6 is the new type of address that is 128-bit number with $2^{128}$ possible  addresses.

IPv4: `128.6.24.78`

IPv6: `2001:4000:A000:C000:6000:B001:412A:8000`

There can be more than one application running on a host, however. They can't interfere with each other, so we need a way to distinguish them. This is done via *port numbers*, which are 16-bit numbers that can be bound to applications that communicate over the network. Some port numbers are reserved for special purposes, such as port 25 for email, port 80 for HTTP, port 443 for HTTPS. Port numbers are like different employees that work at a call center.

Both client and server must identify each other's IP address and port numbers: a 4-tuple: $(S_{IP}, S_{P\\#}, D_{IP}, D_{P\\#})$ This tuple is a *uniquely defined bidirectional connection*.

The OS manages a simple data structure of port numbers to track which are in use, no complexity involved.

## Client-Server Architecture

**Many clients communicate with one always-on server.**

A *server* is a host that is always on and has a permanent IP address that is often public. For large servers, it is often necessary to use server farms that distribute servers geographically for faster communication across the globe.

A *client* is one of many hosts that makes a connection with the server to access data on the server. They might connect intermittently or have dynamic IP addresses. Clients do not communicate with one another directly, instead using the server as a medium.

For example, a website server might listen on a fixed, public IP address at port 80, waiting for a HTTP request. A client might request a webpage from this server and make a connection with its own IP address and port 80. Many clients will all have different connection tuples because of their own unique IP addresses. This way, many clients can make connections without confusing the server.

One domain name can map to different IP addresses. This is how *google.com* can be split across thousands of servers that serve billions of requests every day to the exact same domain. The domain name can be thought of as a contact with saved name that might have many phone numbers.

## DNS

**Domain names are converted to IP addresses through the DNS.**

*DNS* is the *Domain Name System* which acts as a service to resolve IP addresses from a normal alphanumeric domain name, just like a telephone book. The DNS is itself an online service that must be accessed through a network.

The DNS listens on port 53 and the IP address is well-known. A simple DNS might be a large centralized database of names and IP addresses, around 4 billion for IPv4.

However, if this DNS crashes, that would destroy the whole Internet. Also, everybody on the Internet would be trying to access this server, which place tremendous load. It would be a big security issue and a large target for attack, both physical and virtual. Lookup in a server with billions of entries would be slow. Latency for hosts that are physically far away would be greatly increased depending on the server location. Every new host would need to be entered in the same location, which would be slow. *This does not scale!*

Real DNS is implemented as a distributed service across different countries. *Top-level domains* like `.com`, `.org`, `.edu`, are most popular and are the main domain names that people access. Each country also has it's own top-level domain server named after it, like `.de`, `.uk`, `.be`. Many websites nowadays utilize country codes for special names that include the domain name e.g. `youtu.be`.

Second-level servers are the most commonly accessed domain names, like `amazon.com`, `google.com`, etc. These names are separated by periods, from right to left.

Larger domain names like universities can be further namespaced, like `cs.rutgers.edu`. Rutgers University has its own DNS server that manages these additional domain names. The final name that contains the actual data is called the `authoritative` domain.
