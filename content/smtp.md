+++
title = "SMTP Explained"

date = 2022-02-10



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

Email is the most popular way in the world to send large messages. This is a complicated process which requires its own protocol: **SMTP**.

<!-- more -->

## SMTP

The **Simple Mail Transfer Protocol** is how user agents communicate with mail servers. On a high level, email works similarly to physical mail. The mail server is like a post office with mailboxes for each user agent. When a person wants to send a message, they put the envelope in their own mailbox. The mailman user agent then takes the envelope to the post office and puts it in their server mailbox. When a person wants to retrieve a message, their mailman gets all the mail in their post office mailbox and delivers it to the person's personal mailbox. The SMTP post office workers at the server are responsible for moving the mail between mailboxes.

To continue the post office analogy, we can imagine an email being sent between Alice and Bob. Alice lives in Hong Kong and Bob lives in San Juan, so they have different post offices. The workers at the Hong Kong post office see that there is an envelope in Alice's message queue addressed to Hong Kong. They open a TCP connection as the client to the San Juan server and sends the envelope to the San Juan post office. The workers at San Juan now have an envelope, and they see that it is addressed to Bob. They then place the envelope in Bob's mailbox. When Bob wants to check his email, he sends his mailman over to the post office, who returns with the email from Alice.

Let's examine the TCP connection a little more closely. The connection is established on port 25, which is reserved for SMTP. It does some handshaking on the application layer in order to establish the email addresses of the sender and the recipient. The simple exchange goes like this:

- server sends 220 with hostname, client responds with HELO and its own hostname

- server confirms 250 that it's ok

- client tells server MAIL FROM and RCPT TO addresses and the server confirms each with 250

- client sends `DATA` and server confirms with 354 that it's ready to receive mail

- the client sends the entirety of the message, ending with a lone period to finish the message, which the server confirms with 250

- the client can repeat steps 3-5 for any additional messages, then sends `QUIT` for a server 221 response that closes the connection

## vs. HTTP

Both HTTP and SMTP are protocols that transfer files between clients and servers, so there are natural parallels. However, there are also significant differences.

There is a difference in who initiates the connection which defines the two. In HTTP, the *requesting* client initiates the connection, making it a **pull protocol**; you can think of the client "pulling" the data from the server. In SMTP, it is the *sending* client that initiates the connection, making it a **push protocol** where the client "pushes" the data to the other server.

SMTP also has many restrictions. The message can only be in ASCII, so any data that is sent over that is not in ASCII, like multimedia, must be encoded and decoded. HTTP allows any data to be transferred. A result of this is that HTTP data of different types like multimedia must be in its own response message, while SMTP objects are all in the same ASCII message.

There is some peripheral data that SMTP messages can include in a header, just like HTTP, such as the sender, receiver, and the subject of the email. *These are different from commands; this data is part of the actual email, not the protocol!*

## Mail Access

There's a part of the post office analogy that has been ignored so far. How exactly does the mailman get to the post office? After all, this is another client-server connection, so there must also be a protocol that defines this interaction. We can use SMTP for the mailman delivering the envelope, as this is the same kind of "push" that the post office uses to send to the other post office. But this introduces an issue for the person on the other end; if the mailman can only push mail it already has to somewhere else, then how does Bob's mailman get the mail on the server? There are many protocols that can define this interaction, such as **POP3**, **IMAP**, and HTTP.

In 2022, HTTP is the obvious choice for this exchange. After all, everyone accesses email through their web browser, which is making HTTP requests anyway. Why not just use an HTTP request to get the mail from the mail server? And as you might expect, almost every email provider today, from Gmail to Hotmail to Yahoo Mail uses Web-based email.

However, back when "user agent" and "web browser" weren't synonymous, people had applications dedicated to accessing email that couldn't access the web or use HTTP. In those times, **Post Office Protocol—Version 3** and **Internet Mail Access Protocol** were used.

POP3 is a simple and therefore limited protocol. It operates over port 110 and has three steps: authorization, transaction, and update. The post office first authenticates the mailman by asking him for a username and password before he can check your mailbox. Then, the mailman retrieves the envelope from the post office. While the mailman is at the post office, they can also do some operations like marking emails for deletion and getting statistics for your account. When the mailman leaves the post office, the workers throw away all the mail that the mailman marked for deletion.

It's common for users to want to organize emails through folders like Inbox, Junk, etc. This is of course possible through user agent, but POP3 does not allow for this organization in your post office mailbox. IMAP provides this as well as many more features. It is stateful so that the user can keep the folders that it creates at the mailbox. One more important feature is its ability to lazily fetch emails from the mailbox, so the user's entire inbox is not just sent immediately. Imagine if you had to download every email in your inbox in order to access it! I know my connection wouldn't be able to handle my 1k+ unread emails.
