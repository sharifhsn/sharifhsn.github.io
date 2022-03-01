+++
title = "TCP Explained"
date = 2022-02-20

[taxonomies]
categories = ["Internet Technology"]
tags = ["cs352"]
+++

**HTTP** is probably the most visible Internet protocol to end users. It appears (as well as its cousin, **HTTPS**) at the beginning of every URL we use to access the internet. But how does it work?

<!-- more -->

HTTP stands for HyperText Transfer Protocol. It defines the structure of messages that are passed between two programs: a *client* and a *server*. But before we talk about HTTP, let's clarify some vocabulary

## Web Terminology

- **object** - a file that has an associated URL

- **Web page** - a document that consists of multiple objects, typically a base HTML file and other objects. The HTML file will reference other objects through a path.

- **URL** - address consisting of the following parts:

| protocol | hostname           | path name                   |
|:--------:|:------------------:|:---------------------------:|
| http://  | www.someSchool.edu | /someDepartment/picture.gif |

- **Web browser** - an application such as Firefox or Chrome that acts as the client in HTTP

- **Web server** - an application such as Apache that acts as the server in HTTP and houses the Web objects in question

## HTTP Low-Level

Under the hood, HTTP uses TCP to make a reliable connection between the server and client. The client and server both send requests and receive responses from their respective socket interfaces.

When a file is sent, there is no information that the server inherently remembers about the HTTP connection. Because of this, HTTP is considered a **stateless protocol**. This is not always desirable. Sometimes, websites want to remember clients in order to ease use of a website. We will discuss workarounds later.

## Connections

The original HTTP 1.0 protocol was *non-persistent*. This means that every request made between a client and a server had its own connection that was opened and closed every time a request was made. Here's the process:

1. A TCP connection is created between the client and server on port 80.

2. An HTTP request message is sent from the client to the server through the socket.

3. The server receives the message, retrieves the object that it is requesting, and sends the response message to the client through the socket.

4. The server "closes" the TCP connection (TCP will wait until it knows that the client has received the message)

5. The client receives the message and the connection terminates. It will extract the HTML file from the message and initiate a new request for each object referenced within, repeating these steps as needed.

Clearly, this method is inefficient. We are opening a TCP connection for every single request that is made between the same client and server. This inefficiency is mediated slightly by the ability to open serial TCP connections; typical web browsers open 5 to 10 parallel TCP connections. The response time for each connection is \\(2 \\cdot RTT + T_{trans}\\).

HTTP 1.1 introduced *persistent* connections. In step 4, the server doesn't close the TCP connection after sending the response. The entire Web page is sent over the same TCP connection, and the requests can be pipelined for further performance gains. The connection is typically closed on timeout. HTTP/2 further builds on this by interleaving requests and responses in the same connection with a way to preserve ordering on each end.

## Message Format

```http
GET /http HTTP/1.1
Host: sharifhsn.github.io
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```

The `Host` line is technically unnecessary, since the connection is already established by this point, but it's useful for Web proxy caches. The `Connection` line means this connection should be non-persistent. The `User-agent` line specifies the **user agent**, which is the browser type that is making the request to the server. This is used so that the server can send different websites to different browsers. `Accept-language` expresses a preference for content in French if it exists, otherwise give the default.
