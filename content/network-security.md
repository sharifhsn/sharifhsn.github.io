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



## Signatures

A **signature** is a uniquely identified string associated with a person, just like with real signatures. By including a signature with a message, the message can be authenticated and confirmed to have integrity.


