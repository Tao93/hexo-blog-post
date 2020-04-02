---
title: Network security basics
tags: [TLS, Cryptography]
mathjax: true
---

This post mentions elliptic curve for many times, check the post [Elliptic curve cryptography basics](http://tao93.top/2020/01/04/Elliptic%20curve%20group/) for more information.

Everyone knows HTTP is in plaintext and unsecure, and HTTPS is more secure. Just as the lock icon in Chrome address bar shows:

![](http://tao93.top/images/2020/03/30/1585623855.png)

But why is that? Just because of encryption? The reality could be more complicated, yet more interesting. Let's go!

### What is secure communication

First, we should abstractly specify what is secure communication (typically secure remote communication). Assume **Alice (A)** and **Bob (B)** are two characters to communicate remotely, and the meanful contents they send or receive are called **messages**.

There are 3 essential conditions of secure communication

#### Confidentiality

Only the A and B should be able to understand the contents of the transmitted messages. Aparently encryption and decryption are necessary.

#### Message Integrity

Ensure that the messages are not altered, either maliciously or by accident. Digest algorithms (explained laterly) is necessary for integrity.

#### End-Point Authentication

A and B should be able to confirm the identity of the other one, i.e. confirm that the other one is indeed who or what they claim to be.

### Why need secure communication

The Internet is built with numerous routers, switchers, hosts and other devices. Data sent from our PCs or phones would go through so many transit devices, but the hardwares of the Internet doesn't provide secure communication. Some security protocols in network layer or below layers are also unreliable since they're implemented in routers or switchers.

### Encryption

Roughly divided, there're two kinds of Encryption: Symmetric and Asymmetric.

#### Symmetric Encryption

With long history, widely used in wars befoer modern era for thousands of yesrs to pass confidential information. 

Symmectric Encryption is symmectric because A and B use same secret key to encrypt and decrypt the messages:

![](http://tao93.top/images/2020/03/30/1585625689.png)

The biggest inconvenience of symmetric is that A must hand over the secret key to B before any communication, and this is hard for remote communication. In wars, we often see codebooks must be carried to frontlines safely.

Common symmetric encryption algorithms are AES (Advanced Encryption Standard), DES (Data Encryption Standard) etc.

#### Asymmetric Encryption

In Asymmetric Encryption, there are two kinds of keys: public keys and private keys. A has his public key and private key, so does B. Hence there're 4 different keys, and the private keys must be only accessible to the owners.

Asymmetric Encryption are also called public-key encryption.

![](http://tao93.top/images/2020/03/31/1585627448.png)

The most famous asymmetric encryptions are the RSA algorithm and Elliptic curve algorithm. Asymmetric Encryption fulfill the following conditions:

1. both public key and private key could be used to encrypt or decrypt;
2. for a pair of keys, the public key can decrypt anything encrypted with the private key, vice versa;
3. it's very hard to calculate the private key with given public key.

The concrete steps of the RSA algorithm:

1. Choose 2 large primes $$p$$ and $$q$$; the larger the values, the difficult to calculate private key from public key, but also the slower to encrypt and decrypt.
2. Compute $$n = pq$$ and $$z = (p – 1)(q – 1)$$.
3. Choose a number $$e$$, less than $$n$$, that has no common factors with $$z$$.
4. Find a number $$d$$, such that $$(ed – 1 \mod z) = 0$$.
5. Public key is the pair of numbers $$(n, e)$$, private key is $$(n, d)$$
6. For any integer $$m<n$$, calculate $$(m^{e} \mod n)$$ to encrypt or decrypt it with public key, similar for private key.

With given $$n$$, calculate the large primes $$p$$ and $$q$$ is hard, that's why RSA is safe.

The following facts ensures the 2nd condition that asymmetric encryption fulfills:

1. $$(a + b) \mod n = [(a \mod n) + (b \mod n)] \mod n$$.
2.  if $$p$$ and $$q$$ are primes, $$n = pq$$, and $$z = (p – 1)(q – 1)$$, then $$x^y \mod n$$ is the same as $$x^{y \mod z} \mod n$$.

With 1st of the above facts, there is $$[a^d \mod n] = [(a \mod n)^d \mod n]$$, then there is:

$$[(m^e \mod n)^d \mod n] = [m^{ed} \mod n] = [m^{ed \mod z} \mod n] = (m \mod n) = m$$

With asymmetric encryption, A and B publically publish their public keys. A could encrypt messages by using B's public key before send them to B. After receive A's encrypted message, B could decrypt the encrypted messages to get the original contents:

![](http://tao93.top/images/2020/03/31/1585628013.png)

Except A and B, no one could decrypt the encrypted messages as long as their private keys are safe. Seems perfect! However, there're man in the middle attacks!

Imagine a hacker called Eve controlled a router between A and B, and is able to see all data passed between A and B. Then when A and B exchange their public keys, Eve could manipulate the router to replace A and B's public keys with Eve's public key, thus A and B would encrypt messages with Eve's public key, and all encrypted messages are decryptable to Eve. Yet A and B might be unware of this:

![](http://tao93.top/images/2020/03/31/1585630134.png)

Aparently, the critical point to avoid the MITM (man in the middle) attack is making sure the public key exchange cound't be tampered. In real world, certificates and certificate authority (CA) are invented to solve this.

#### Pros and cons of Asymmetric and Symmetric

**Symmetric**: simple and fast, a shared secret key need to be transitted..

**Asymmetric**: complex and slow, only need public keys exchange.

In real Internet, they're working together in HTTPS.

Besides, digital signature was based on asymmetric encryption.

#### Digest

Digest algorithms are also called hash functions. A digest algorithm maps input of any length to an output with fixed length. 

A good digest algorithm fulfills that, for any input $$x$$, it's hard to find another input $$y$$ such that digest of $$y$$ is same as $$x$$.

Common digest algorithms are MD5 and SHA:

![](http://tao93.top/images/2020/03/31/1585680049.png)

Digest is also used to verify the integrity of large file you downloaded form Internet.

#### Digital signature

Signature should be verifiable and nonforgeable.

A real world signature, to some extent, can prove that someone certified a document, on condition that no one can imitate handwriting of others.

![](http://tao93.top/images/2020/03/31/1585680509.png)

Digital signature is based on asymmetric encryption. Assume you keep your private key safe (only accessible to yourself). For any message, you calculate its digest and encrypted the digest with your private key, then the message and the encrypted digest form a digital signature by you:

![](http://tao93.top/images/2020/03/31/1585680879.png)

Anyone holds your real public key can verify the digital signature, and no one can forge your signature.

#### Certificate and certificate chain

You definitely want everyone who holds your public key is really your public key. To achieve this, we need certificate.

Let's look at the real world network communication. There are numerous servers, yet there are more clients like our PCs and phones. Certificates would help us (clients) to get the real public keys of servers, and confirm the identify of a server.

Certificates are public, encoded in [X.509](https://en.wikipedia.org/wiki/X.509) format, and contain (but not only) the following information:

1. basic info of the certificate owner, i.e. the server.
2. public key of the server.
3. basic info and signature of the issuer of the certificate.

An example of certificate owned by wikipedia:

![](http://tao93.top/images/2020/03/31/1585682422.png)

In the world, there are some entities called certificate authorities (CAs), they issue certificates to other servers. Meanwhile they also own their certificates. 

A certificate is called self-signed if the issuer is the owner itself. Some CAs own self-signed certificates, and they're called root CAs. The self-signed certificates owned by root CAs are called root certificates.

Root certificates are powerful because many of them are embedded in our client devices when they were made in factory (we can also install certificates to our devices, i.e. trust the installed certificates). Thus when a client communicates with a root CA, the client can confirm identity of the server by requesting digital signature of random message from the server.

Common root CAs are: Comodo, Symantec, GoDaddy, GlobalSign and DigiCert etc.

But how about ordinary servers? How to identify them? Certificate chains would help us.

Assume there is a certificate chain with 3 certificates:

1. the 3rd certificate is a root certificate, owned and issued by a root CA; 
2. the 2nd is called intermediate certificate, owned by a CA and issued by the root CA;
3. the 1st certificate (end entity certificate) is owned by a ordinary server and issued by the intermediate CA.

![](http://tao93.top/images/2020/03/31/1585692503.png)

If the root certificate is trusted by client, then the client can verify the signature in intermediate certificate, and confirm whether the intermediate certificate is really issued by the root CA. If so, then the public key (of intermediate CA) in intermediate certificate would be able to verify the 3rd certificate. Overall, the client is able to verify the whole certificate chain.

A screenshot of real certificate chain:

![](http://tao93.top/images/2020/03/31/1585705776.png)

But verifying the certificate chain is not enough. Furthuremore, the client could request a signature of random message from the ordinary server, and verify the signature with public key from the end entity certificate, thus finally confirm identity of the oridinary server.

#### Rough steps to establish secure communication

The following are the rough steps to establish secure communication. They don't reflect the real things in HTTPS, but provide the abstract frame.

1. Server send certificate chain to Client.
2. Client verify the certificate chain with its lcoally trusted certificates, then send random message to server.
3. Server sign the message, and send signature back to client.
4. Client verify the signature, then send some params from which both side could generate symmtric encryption key, of course the params are encrypted with Server's public key.
5. Both sides generate symmtric key, with which then they could communicate securely and quickly.
6. Sometimes, the client has to provide credentials (encrypted with symmtric key) for authentication.

![](http://tao93.top/images/2020/03/31/1585693925.png)

#### Certificate pinning

Only trust a specific certificate, or certificates issued by owner of that specific certificate.

Our Android app accept certificates as asset resources to enable certificate pinning. Concretely, we have a customized [X509TrustManager](https://docs.oracle.com/javase/7/docs/api/javax/net/ssl/X509TrustManager.html). The customized X509TrustManager read certificates from asset resources, override the `checkServerTrusted` method to achieve the goal, and would be set into [SSLSocketFactory](https://docs.oracle.com/javase/8/docs/api/javax/net/ssl/SSLSocketFactory.html).

For security, we should only add end entity certificate to the asset resources, or other certificates issued by intermediate CAs would be trusted by our App.

#### ECDH (Elliptic curve Diffie-Hellman) Key Exchange algorithm

Key exchange algorithms are to let A and B have shared secret key for symmetric encryption.

There are 2 kinds of key exchange algorithms: RSA based and Diffie-Hellman algorithm. 

RSA based Key exchange algorithm is simpler, in which the client would generate the symmtric key by itsefl, then encrypt it with server's public key and send it to server. 

While the Diffie-Hellman key exchange algorithm is more complex. The following is a vivid demonstration of the Diffie-Hellman algorithm:

![](http://tao93.top/images/2020/03/31/1585701668.png)

In above diagram: 

1. secret colors are private keys;
2. secret colors + common paint = public keys (it's hard to get secret colors from common paint and public keys)
3. A's public key + B's secret color = A's secret color + B's public key, and that's the symmetric key.

Therefore, in Diffie-Hellman algorithm, A and B could calculate the symmetric key respectively. 

Original Diffie-Hellman algorithm is based on exponential modular. Assume $$p$$ is a prime and $$k$$ is a [primitive root](https://en.wikipedia.org/wiki/Primitive_root_modulo_n) modulo $$p$$ and both of them are public. Private keys are random numbers $$d_A$$ and $$d_B$$, public keys are $$[k^{d_A} \mod p]$$ and $$[k^{d_B} \mod p]$$, and the secret key for symmectric encryption is $$k^{d_A d_B} \mod p$$.

The ECDH Key Exchange algorithm is enhanced by replacing exponential modular with elliptic curve [group](https://en.wikipedia.org/wiki/Group_(mathematics)). 

#### The real TLS handshake

HTTPS = HTTP + TLS (deprecated predecessor is SSL). 

In most programming languages, socket acts as API between Transport layer and Application layer. With TLS, a new abstract sublayer is added:

![](http://tao93.top/images/2020/03/31/1585695667.png)

Although TLS is in the application layer, from the developer's perspective it's like a transport-layer protocol.

The following is an example of TLS handshake procedure. In the example the key exchange algorithm is not RSA but ECDH, hence the following diagram and steps couldn't exactly represents TLS handshake using RSA as key exchange algorithm.

Diagram of TLS handshake using ECDH key exchange algorithm:

![](http://tao93.top/images/2020/03/31/1585697894.png)

##### 1. Client Hello (sent by client)

Contains all the cipher suits that the client could support, a random number. A cipher suit example is like the following:

`TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384`

where

``` 
TLS: protocol name
ECDHE: key exchange algorithm, ECDHE = Elliptic curve Diffie-Hellman Ephemeral
ECDSA: signature algorithm, ECDSA = Elliptic curve digital signature algorithm
AES_256_CBC: symmetric encryption algorithm
SHA384: digest algorithm
```

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585698860.png)

##### 2. Server Hello (sent by server)

Contains the server suit that the server selected, and a random number.

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585698960.png)

##### 3. Certificate (sent by server)

Contains the certificate chain. 

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585699156.png)

##### 4. Server Key Exchange (sent by server)

For ECDHE as key exchange algorithm, some necessary ECDH params are contained in this TLS packet. Another critical part is a signature of some important data so far transferred (including the random numbers and ECDH params). With this signature, the client would be able to identify the server. Note the signature algorithm should be consistent with asymmetric encryption algorithm of server certificate.

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585699514.png)

In above picture, the named curve `x25519` is a concrete elliptic curve; the `PubKey` is the public key in ECDH key exchange algorithm. A and B need to exchange them to generate symmtric key. Note the `PubKey` is not the public key contained in certificate of the server.

##### 5. Server Hello Done (sent by server)

Trivial information

##### 6. Client Key Exchange (sent by client)

For ECDHE as key exchange algorithm, this TLS packet contains `PubKey`, i.e. public key of client for ECDH key exchange algorithm.

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585702469.png)

##### 7. Change Cipher Spec (sent by both)

Trivial information, tell the other side I'm going to use symmtric key for the following communication.

##### 8. Encrypted Handshake Message (sent by both)

Encrypted the handshake data with symmtric key and send to other side, act as verification.

After the above 8 steps, TLS handshake is over, A and B could securely communicate.


### References

1. [Computer network a top down approach](https://eclass.teicrete.gr/modules/document/file.php/TP326/%CE%98%CE%B5%CF%89%CF%81%CE%AF%CE%B1%20(Lectures)/Computer_Networking_A_Top-Down_Approach.pdf);
2. [ECDHE server key exchange explanation](https://crypto.stackexchange.com/questions/66104/public-key-on-server-key-exchange-generation-vs-public-key-on-servers-certifica/66107#66107?newreg=dd00f58ac00d4805a945c2448f6bb314);
3. [TLS handshak explanation](https://blog.catchpoint.com/2017/05/12/dissecting-tls-using-wireshark/);
4. [TLS handshak explanation](https://razeencheng.com/post/ssl-handshake-detail);
5. [Elliptic Curve Cryptography: a gentle introduction](https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/).