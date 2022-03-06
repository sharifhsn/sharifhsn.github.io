+++
title = "FTP Explained"

date = "2022-02-07"



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

Before we had Google Drive, users needed a way to access files from other people quickly and easily. The method for that was **FTP**.

<!-- more -->

## FTP

The point of the **File Transfer Protocol**, was, as is obvious, to transfer files between two hosts. Like HTTP, FTP operates between a server and client, each of which have their own filesystem that they want to access. It also creates a TCP connection which has certain verification steps to establish a connection that allows the client to copy files stored in the server or vice versa.

Unlike HTTP however, FTP operates using *two* parallel connections, the **control connection** and the **data connection**. As the names imply, the control connection is used for verification and command information such as which file to get in which directory, and the data connection is the connection used to send the files. This is called an *out-of-band* connection, unlike HTTP and SMTP, which have the request information in a header in the same connection and are therefore *in-band*. 

The session begins with a control connection initiated by the client to identify the user and commands. The server receives this information, and if it approves, it initiates a data connection with the client. Like HTTP 1.0, this is a non-persistent connection that is only open for the passing of a single file.

The nature of FTP authentication means that FTP must maintain state about the user account, unlike HTTP which is totally stateless by design. This introduces constraints on the number of simultaneous FTP connections.

## Commands

FTP commands work like HTTP commands in request headers. They are ASCII and each lines ends with `\r\n`. Some example commands are:

- `USER username`

- `PASS password`

- `LIST`: works like the `ls` command for the remote directory, sent over data connection

- `RETR filename`: retrieves the file from the remote directory, over data

- `STOR filename`: stores the file from current directory into remote, over data

Every command has a corresponding reply with a status code, similar to HTTP status codes.
