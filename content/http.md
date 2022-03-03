+++
title = "HTTP Explained"
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

The `Host` line is technically unnecessary, since the connection is already established by this point, but it's useful for Web proxy caches. The `Connection` line means this connection should be non-persistent. The `User-agent` line specifies the **user agent**, which is the browser type that is making the request to the server. This is used so that the server can send different websites to different browsers. `Accept-language` expresses a preference for content in French if it exists, otherwise give the default. In general, the header lines exist to give information that is necessary for the request.

The HTTP response is different than the request. An example response to this message might be:

```http
HTTP/1.1 200 OK
Connection: close
Date: Tue, 09 Aug 2011 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 09 Aug 2011 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html
```

The status line at the top tells us some similar information, like the protocol we are using. The `200 OK` is our status code for a good response. There are other status codes:

- 200 OK: request succeeded, information returned.

- 301 Moved Permanently: the address is moved, for example when a website links to its www address; there will be a location header telling the client where the new place is to go

- 400 Bad Request: request is written poorly and could not be understood

- 505 HTTP Version Not Supported: self-explanatory lol

There's tons of valid header lines we can use. The HTTP specification allows for many header lines to be used from various software. Writing a correct HTTP implementation isn't easy.

## Cookies

I mentioned before that HTTP is stateless, which means that the server does not preserve any information about the client after the connection ends. However, websites really want ways to identify users for operations such as logins. In order to facilitate state, websites use **cookies**. You might have heard of them when you log in to a website and it asks you to accept cookies.

The typical creation method of a cookie might look like this:

1. A client makes a request to the server for the first time.

2. The server generates an identification number for the client.

3. The server sends a HTTP response to the client with a cookie in the header: `Set-cookie: 1678`

4. The client browser stores the cookie in a special file and sends a request to the server with the same cookie number.

5. Now, whenever the client requests the server, it will include a cookie in the header which the server will use.

## Caching

You might have heard of a **proxy server** before. These servers are usually used by ISPs like your home ISP and your university ISP. These servers contain frequently-accessed data from several origin servers and are located much more closely to the client.

These servers are extremely helpful in improving response times for the client as well as reducing traffic for the server. The close location and LAN nature of proxy servers mean that the request/response time between clients and servers are quick. And since the origin server is not accessed as often, its burden of traffic is significantly reduced.

The delay in fetching a response from the Internet tends to be dominated by the time for the institutional server receiving a response from the Internet. However, if there is heavy traffic on the *access link* between the institutional server and the Internet router, that can slow down speeds to minutes per request, which is obviously unacceptable.

$$ requestRate \\cdot requestSize / speed = trafficIntensity $$

The intensity between client and LAN is usually negligible, but the access link between LAN and Internet dominates. With a Web cache, hit rate is generally between 0.2 and 0.7.

$$ cacheHit \cdot LANspeed + (1 - cacheHit) \cdot internetSpeed$$

The traffic intensity stops being an issue and Internet delay dominates again once a cache is introduced. It's less expensive and faster than increasing access link speed.
