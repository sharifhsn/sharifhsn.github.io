+++
title = "Network Security"

date = 2022-04-22



[taxonomies]

categories = ["Internet Technology"]

tags = ["cs352"]

+++

The internet has taken over the world, but with it have come malicious actors. People want to spy on you, scam you, steal from you, or harm you through the internet. Through **network security**, we can shield potential victims from at least some of these harms.

<!-- more -->

## Attacks

Attacks on a network can be performed *passively*, for example in eavesdropping. The network cannot tell the difference between this passive attack and ordinary traffic because no more traffic is being injected.

*Active* attacks, by contrast, are done by changing the kind or amount of traffic. This typically manifests as a **DoS** **(Denial of Service)** attack. This is usually a significant increase in traffic which shuts down the network due to congestion. If only one host is attacking the network, then the network can easily shut a particular malicious host out. DoS attacks are usually distributed, hence **DDoS** **(Distributed Denial of Service)**.

Teardrop attacks are a modern form of DDoS. Instead of sending lots of useless traffic, all they do is open a new connection with a socket. Even though this is a small operation, when done on large scale, it can tax the memory of a server. We mentioned previously that in order to compensate for fragmentation, routers will keep a memory buffer to store fragments. This can be exploited by malicious packets that are all fragments which will run out the memory of the router.

The three main aspects of network security are **authentication**, **confidentiality**, and **integrity**.

- Authentication: How do you prove that someone is who they say they are?

- Confidentiality: How can you hide data from prying eyes?

- Integrity: How can you prove that data has not been altered midstream?

In order to protect these, we need a **security model**.

1. An algorithm to ensure security that must be well-known.

2. Trusted parties know how to generate secret information.

3. Distribute that information through a secure method.

4. Specify a protocol for trusted parties to decrypt data.

## Encryption

**Encryption** or **cryptography** is the encoding of a message in such a way that only the communicating parties can interpret it. That way, if a malicious party tries to read the message, they will be unable to understand it because they cannot decrypt it.

Encryption has two sides, the encryption algorithm and the decryption key. The algorithm should be publicly available in order to prove that it works. When you feed the decryption key into the algorithm, it will spit out a string associated with only that key.

The secret key itself should be long enough that the algorithm cannot easily be broken, but also short enough so that it can be transmitted easily. Fun fact: before minimum password lengths were instituted, the most common password was "god".

The most basic kind of encryption algorithm is a substitution cipher. By simply replacing letters in a message with another letter with a known mapping, a message can be encoded and decoded. However, it is relatively trivial to break this cipher because the English language has patterns that can identify certain letters better than others.

You can increase the complexity of this algorithm through polyalphabetic encryption, where the cipher itself changes after every letter. Famously, this is how the Enigma cipher used by the Nazis during World War II worked.

Transposition ciphers will change the *order* of the words as well. This might seem like a really good cipher, unfortunately it can similarly be broken by looking at the structure of language.

In order to actually encrypt data, real-world encryption algorithms use a combination of these techniques.

But how do we generate the decryption key? There needs to be one for every user, but they need to be as random as possible and nigh impossible to crack, so we can't use plaintext passwords.

We can use *pseudorandom number generators* in order to create a key. A pseudorandom number generator is an algorithm which, starting with some seed, will produce a string of numbers based on that seed which *appear* to be random. The seed will typically based on some environmental noise that can't be easily predicted, such as the time of day at which the algorithm runs. The generator should be *highly* sensitive to changes in seed.

## Symmetric Key

Using the same key, two parties should be able to both encrypt and decrypt the message. By applying the key to the message, the message should be encrypted, then decrypted.

There are two main kinds of symmetric ciphers: *stream ciphers*, and *block ciphers*. As the name implies, stream ciphers do bitwise operations on the message and the key in order to get the ciphertext. The bitwise operations can be symmetric, like ⊗. One example of a stream cipher is RC4 which can be used in SSL.

Block ciphers have a 1-to-1 mapping of certain blocks of \\(k\\) bits to other blocks, typically 64 bits. This is a good method, but the problem is that the number of mappings balloons massively: in general, it is \\(2^k!\\), which is absolutely massive! It is both exponential *and* factorial. To simulate such a table, you can use a pseudorandom function.

## DES

The **Data Encryption Standard** is the US encryption standard. It uses a 56-bit symmetric key for 64-bit plaintext input, and it is a block cipher. It is fairly secure for small attacks, but it can be brute force decrypted in less than a day. Its advantage is that there is no analytic exploit in the standard.

In order to make DES more secure, we can increase the number of keys to 3, making **3DES**. The message is encrypted, decrypted, then encrypted again when run through all three keys. Effectively, there are \\(56 × 3 = 168\\) bits to crack, which is much stronger than 56.

## Replay Attacks

This method of symmetric key is vulnerable to a certain kind of attack: the **replay attack**. Let's say I encrypt a message and mail it to my boss. My boss decrypts the message, and sends me back a confidential message, so we can pass information to each other. But my boss doesn't know my address, he only knows that he can trust me because the message is encrypted using our key.

What if a malicious hacker somehow obtains my encrypted message? If she sends that message to my boss, then my boss will email the hacker back with the confidential information because he thinks that only I have the ability to encrypt messages in that way.

The way to protect against this is by using a **NONCE** value, which is a **N**umber used only **ONCE**. Let's say I write a random number on my message every time I send a message to my boss. The hacker obtains my message and sends the same message later, with the same random number. My boss knows that the NONCE value cannot be the same and therefore does not trust the hacker with our confidential information.

The NONCE doesn't have to be a random number. Many times, it can be a challenge query like asking for my mother's maiden name. Having multiple challenges that are cycled through adds an extra layer of protection.

## RSA

Nowadays, almost all cryptography is done by having two keys: a **public key**, and a **private key**. The public key is used to *encrypt* messages, and private key is used to *decrypt* messages. The public key is, as might be obvious, publicly available. Someone who wants to send a secure message to me can use my public key to encrypt it. However, only I can use my own private key to decrypt it. Importantly, *you should not* be able to determine the private key by analyzing the public key. They should be wholly different in nature.

**RSA** is the most commonly used implementation of this kind of cryptography. To encrypt a message, take the message and raise it to the power of the public key, `mod` some \\(n\\). To decrypt said bit pattern, raise it to the power of the private key, again `mod` the same \\(n\\).

Authentication can be done using the passing of challenge messages. A new secret key will be created which is encrypted and decrypted in order to encrypt the data.

One problem with RSA is that it's really expensive and slow computation-wise, which is good for security but bad for convenience. In practice, most uses of RSA encryption only use the actual algorithm to encrypt some secret key, which is then used as a simpler cipher for the data between the two parties.

## Hash

**Hashing** is the process of creating a unique-ish value from some data that can be used to verify that data as a signature. It should be fast, not easily irreversible, and not have collisions i.e. no two hashes should be the same.

The two most common hash functions are **MD5** and **SHA-1**, which are 128 bits and 160 bits, respectively.


