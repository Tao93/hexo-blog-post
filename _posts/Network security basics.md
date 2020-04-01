---
title: Network security basics
tags: [TLS, Cryptography]
mathjax: true
---

Everyone knows HTTP is in plaintext and unsecure, and HTTPS is more secure. Just as the lock icon in Chrome address bar shows:

![](http://tao93.top/images/2020/03/30/1585623855.png)

But why is that? Just because of encryption? The reality could be more complicated, yet more interesting. Let's go!

### What is secure communication

First, we should abstractly specify what is secure communication (typically remote communication). Assume **Alice (A)** and **Bob (B)** are two characters to communicate remotely, and the meanful contents they send or receive are called **messages**.

There are 3 essential conditions of secure communication

#### Confidentiality

Only the A and B should be able to understand the contents of the transmitted message. Absolutely encryption and decryption are necessary.

#### Message integrity

Ensure that the messages are not altered, either maliciously or by accident. Digest algorithms (explained laterly) is necessary for integrity.

#### End-point authentication

A and B should be able to confirm the identity of the other one, i.e. confirm that the other one is indeed who or what they claim to be.

### Why need secure communication

The Internet is built with numerous routers, switchers, hosts and other devices. Data sent from our PCs or phones would go through so many transit devices, but the hardwares of the Internet doesn't provide secure communication. Some security protocols in network layer or below layers are also unreliable since they're implemented in routers or switchers.

### Encryption

Roughly divided, there're two kinds of Encryption: Symmetric and Asymmetric.

#### Symmetric Encryption

With long history, widely used in wars befoer modern era for thousands yesrs to pass confidential information. 

Symmectric Encryption is symmectric because A and B use same secret key to encrypt and decrypt the messages:

![](http://tao93.top/images/2020/03/30/1585625689.png)

The biggest inconvenience of symmetric is that A must be handed over the secret key to B before any communication, and this is hard for remote communication. In wars, we often see codebooks must be carried to frontlines safely.

Common symmetric encryption algorithms are AES (Advanced Encryption Standard), DES (Data Encryption Standard) etc.

#### Asymmetric Encryption

In Asymmetric Encryption, Keys are have two kinds: public keys and private keys. A has his public key and private key, so does B. Hence there're 4 different keys, and the private keys must be only accessible to the owners.

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

The following facts ensures the 2rd conditions that RSA fulfills:

1. $$(a + b) \mod n = [(a \mod n) + (b \mod n)] \mod n$$.
2.  if $$p$$ and $$q$$ are primes, $$n = pq$$, and $$z = (p – 1)(q – 1)$$, then $$x^y \mod n$$ is the same as $$x^{y \mod z} \mod n$$.

With 1st of the above facts, there is $$[(a \mod n)^d \mod n] = [a^d \mod n]$$, then there is:

$$(m^e \mod n)^d \mod n = m^{ed} \mod n = m^{ed \mod z} \mod n = m \mod n = m$$

With asymmetric encryption, A and B publically publish their public keys. A could encrypt messages by using B's public key before send them to B. After receive A's encrypted message, B could decrypt the encrypted messages to get the original contents:

![](http://tao93.top/images/2020/03/31/1585628013.png)

Except A and B, no one could decrypt the encrypted messages as long as their private keys are safe. Seems perfect! However, there're man in the middle attacks!

Imagine a hacker called Eve controlled a router between A and B, and is able to see all data passed between A and B. Then when A and B exchange their public keys, Eve could manipulate the router to replace A and B's public keys with Eve's public key, thus A and B would encrypt messages with Eve's public key, and all encrypted messages are decryptable to Eve. Yet A and B might be unware of this:

![](http://tao93.top/images/2020/03/31/1585630134.png)

Aparently, the critical point to avoid the MITM (man in the middle) attack is making sure the public key exchange cound't be tampered. In real world, certificates and certificate authority (CA) are invented to solve this.

#### Pros and cons of Asymmetric and Symmetric

**Symmetric**: simple and fast, but need to make sure the same key is only accessible to A and B.

**Asymmetric**: complex and slow, only needs the public key exchange.

In real Internet, they're working together in HTTPS.

Besides, digital signature was based on asymmetric encryption.

#### Digest

Digest algorithms are also called hash functions. A digest algorithm maps input of any length to an output with fixed length. 

A good digest algorithm fulfills that, for any input $$x$$, it's hard to find another input $$y$$ such that digest of $$y$$ is same as $$x$$.

Common digest algorithms are MD5 and SHA:

![](http://tao93.top/images/2020/03/31/1585680049.png)

Digest is also used to verify the integrity of large file you downloaded form Internet.

#### Digital signature

A real world signature, to some extent, can prove that someone certified a document. This requires that no one can imitate handwriting of others.

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

In the world, there are some entities called certificate authorities (CAs), and some of them are called root CAs. The certificates owned by root CAs are called root certificates.

A certificate is called self-signed if the issuer is the owner itself. Actually root certificates are self-signed.

Root certificates are powerful because many of them are embedded in our client devices when they were made in factory (we can also install certificates to our devices, i.e. trust the installed certificates). Thus when a client communicates with a root CA, the client can confirm identity of the server by requesting digital signature of random message from the server.

Common root CAs are: Comodo, Symantec, GoDaddy, GlobalSign and DigiCert etc.

But how about ordinary servers? How to identify them? Certificate chains would help us.

In a certificate chain with 3 certificates:

1. the 1st certificate is a root certificate, owned and issued by a root CA; 
2. the 2nd is called intermediate certificate, owned by a CA and issued by the root CA;
3. the 3rd certificate (end entity certificate) is owned by a ordinary server and issued by the intermediate CA.

![](http://tao93.top/images/2020/03/31/1585692503.png)

If the root certificate is trusted by client, then the client can verify the signature in intermediate certificate, and confirm whether the intermediate certificate is really issued by the root CA. If so, then the public key (of intermediate CA) in intermediate certificate would be able to verify the 3rd certificate. Overall, the client is able to verify the whole certificate chain.

But verifying the certificate chain is not enough. Furthuremore, the client could request a signature of random message from the ordinary server, and verify the signature with public key from the end entity certificate, thus finally confirm identity of the oridinary server.

#### Rough steps to establish secure communication

The following is the rough steps to establish secure communication. They don't reflect the real things in HTTPS, but provide the abstract skeleton.

1. Server send certificate chain.
2. Client verify the certificate chain with its lcoally trusted certificates, then send random message to server.
3. Server sign the message, and send signature back to client.
4. Client verify the signature, then send some params from which both side could generate symmtric encryption key, of course the params are encrypted with Server's public key.
5. Both sides generate symmtric key, with which then they could communicate securely and quickly.
6. Usually, the client has to provide credentials (encrypted with symmtric key) for authentication.

![](http://tao93.top/images/2020/03/31/1585693925.png)

#### Certificate pinning

Only trust a specific certificate, or certificates issued by owner of that specific certificate.

Our Android app accept certificates as asset resources to enable certificate pinning. Concretely, we have a customized [X509TrustManager](https://docs.oracle.com/javase/7/docs/api/javax/net/ssl/X509TrustManager.html). The customized X509TrustManager read certificates from asset resources, override the `checkServerTrusted` method to achieve the goal, and would be set into [SSLSocketFactory](https://docs.oracle.com/javase/8/docs/api/javax/net/ssl/SSLSocketFactory.html).

For security, we should only add end entity certificate to the asset resources, or other certificates issued by intermediate CAs would be trusted by our App.

#### ECDH (Elliptic curve Diffie-Hellman) Key Exchange algorithm

There are 2 kinds of key exchange algorithms: RSA based and Diffie-Hellman algorithm. 

RSA based Key exchange algorithm is simpler, in which the client would generate the symmtric key by itsefl, then encrypt it with server's public key and send it to server. 

While the Diffie-Hellman key exchange algorithm is more complex. The following is a vivid demonstration of the Diffie-Hellman algorithm:

![](http://tao93.top/images/2020/03/31/1585701668.png)

In above diagram: 

1. secret colors are private keys;
2. secret colors + common paint = public keys (it's hard to get secret colors from common paint and public keys)
3. A's public key + B's secret color = A's secret color + B's public key, and that's the symmetric key.

Therefore, in Diffie-Hellman algorithm, A and B could calculate the symmetric key respectively. 

Original Diffie-Hellman algorithm is based on exponential modular. Assume $$p$$ is a prime and $$k$$ is a primitive root modulo $$p$$ and both of them are public. Private keys are random numbers $$d_A$$ and $$d_B$$, public keys are $$k^{d_A} \mod p$$ and $$k^{d_B} \mod p$$, and the secret key for symmectric encryption is $$k^{d_A d_B} \mod p$$.

The ECDH Key Exchange algorithm is enhanced by replacing exponential modular with elliptic curve group. 

#### The real TLS handshake in HTTPS

HTTPS = HTTP + TLS (deprecated predecessor is SSL). 

In most programming languages, socket acts as API between Transport layer and Application layer. With TLS, a new abstract sublayer is added:

![](http://tao93.top/images/2020/03/31/1585695667.png)

Although TLS is in the application layer, from the developer's perspective it's like a transport-layer protocol.

The following is an example of TLS handshake procedure. In the example the key exchange algorithm is not RSA, hence the following diagram and steps couldn't exactly represents TLS handshake using RSA as key exchange algorithm.

Diagram of TLS handshake using ECDH key exchange algorithm:

![](http://tao93.top/images/2020/03/31/1585697894.png)

##### 1. Client Hello

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

##### 2. Server Hello

Contains the server suit that the server selected, and a random number.

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585698960.png)

##### 3. Certificate

Contains the certificate chain. 

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585699156.png)

##### 4. Server Key Exchange

For ECDHE as key exchange algorithm, some necessary ECDH params are contained in this TLS packet. Another critical part is a signature of some important data so far transferred (including the random numbers and ECDH params). With this signature, the client would be able to identify the server.

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585699514.png)

In above picture, the named curve `x25519` is a concrete elliptic curve; the `PubKey` is the public key in ECDH key exchange algorithm. A and B need to exchange them to generate symmtric key.

##### 5. Server Hello Done

Trivial information

##### 6. Client Key Exchange

For ECDHE as key exchange algorithm, this TLS packet contains PubKey, i.e. public key of client for ECDH key exchange algorithm.

Screenshot of Wireshark:

![](http://tao93.top/images/2020/03/31/1585702469.png)

##### 7. Change Cipher Spec

Both A and B have this step.

Trivial information, tell the other side I'm going to use symmtric key for the following communication.

##### 8. Encrypted Handshake Message

Both A and B have this step.

Encrypted the handshake data with symmtric key and send to other side, act as verification.

After the above 8 steps, TLS handshake is over, A and B could securely communicate.

### More about Elliptic curve cryptography

We saw so many elliptic curve in above sections. You might be interested in what's an elliptic curve. First, elliptic curves are not conic section we learnt in high school.

#### [Group](https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FGroup_%28mathematics%29)

Before the elliptic curve, we should talk about [Group](https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FGroup_%28mathematics%29).

A set $$G$$ is a group if a binary operation could be defined on it, and 4 conditions are met. Usually we denote the operation as addition and denoted as $$\oplus$$, then the 4 conditions are:

1. **Closure**: $$m \oplus n$$ is always in $$G$$ as long as $$m$$ and $$n$$ in $$G$$.
2. **Associativity**: for any $$m$$, $$n$$ and $$r$$ in $$G$$, $$(m \oplus n) \oplus r = m \oplus (n \oplus r)$$.
3. **Identity element**: there is an element $$\circledcirc$$ in $$G$$ such that $$m \oplus \circledcirc = \circledcirc \oplus m = m$$ for any $$m$$ in $$G$$.
4. **Inverse element**: for any $$m$$ in $$G$$, there is an element $$n$$ in $$G$$ such that $$m \oplus n = \circledcirc $$, and we say $$n = \ominus m$$.

A group is a [Abelian group](https://en.wikipedia.org/wiki/Abelian_group) if the follwoing condition is also met:

**commutation**: for any $$m$$ and $$n$$ in $$G$$, there is $$m \oplus n = n \oplus m$$

Aparently the set of all real numbers $$R$$ could be an Abelian group with the arithmetic adding operation.

#### Scalar multiplication

Scalar multiplication measn multiplication between a natural number and an element of a group:

For any natural number $$d$$, and any element $$m$$ of group $$G$$.

1. if $$d=0$$, $$dm = \circledcirc$$.
2. otherwise, $$dm$$ means adding $$m$$ to itself for $$d$$ times.

#### Finite group

A finite group is a group with limited elements. For any element $$m \ne \circledcirc$$ in a group $$G$$, if you repeatly add $$m$$ to itself, you would get $$G$$. I.e. there is a number $$d$$  such that $$dm = \circledcirc $$.

Why? Lets say the group has $$n$$ elements, then the list $$m, 2m, 3m, ..., (n+1)m$$ would have same elements in it. Assume they are $$xm = ym$$, then you see $$(x-y)m = \circledcirc$$.


#### Continuous elliptic curve

A elliptic curve could be defined as:

$$y^2 = x^3 + ax + b$$

Some examples of elliptic curves are like:

![](http://tao93.top/images/2020/01/04/1578180839.png)

equation for red line is $$y^{2}=x^{3}+4x+4$$, equation for purple line is $$y^{2}=x^{3}-3x$$

### Define group on the elliptic curves

First, the set consists of all points in a elliptic curve. 

The Identity element $$\circledcirc$$ would be a little special, it's defined as the infinitely far points of the curve.

Now, lets define the operation. For any points $$m$$ and $$n$$ in the curve:

1. if $$m = n$$ and $$y_m = 0$$, then $$m \oplus n = \circledcirc $$
2. if $$m = n$$ and $$y_m \ne 0$$, then $$\ominus (m \oplus n)$$ is the other intersection of the curve and the line tangent to the curve in $$m$$.
3. if $$m \ne n$$ and $$x_m = x_n$$, then $$m \oplus n = n \oplus m = \circledcirc $$
4. if $$m \ne n$$ and $$x_m \ne x_n$$, then $$\ominus (m \oplus n)$$ is the 3rd intersection of the curve and the line cross $$m$$ and $$n$$.

From the 3rd and 4th cases above, we know the relation between $$m + n$$ and $$m$$, $$n$$ would be like: 

![](http://tao93.top/images/2020/01/04/1578188228.png)

It can be proved that such a group is an Abelian group.

#### Discrete Elliptic curve

Previously the elliptic curves consist of points with coordinates of real numbers. But now we restrict the curve upon a range of integers betwen 0 and $$P-1$$, where $$P$$ is a prime.

$$y^2 \equiv x^3 + ax + b \mod P$$

When 

$$4a^3 + 27b^2 \not\equiv 0 \mod P$$

Where $$P$$ is a prime number, $$x$$, $$y$$, $$a$$ and $$b$$ are integers, and $$0\le x, y\le P-1$$.

Here is an example graph of $$y^2 \equiv x^3 - 7x + 10 \mod 19$$:

![](http://tao93.top/images/2020/01/05/1578243411.png)

Note that most of the points in the above graph are symmetrical over the line $$y=P/2$$ except the points on line $$y=0$$.

#### Group on the elliptic curve over finite fields

To define a group, firstly we need to define the plus operation on the elliptic curve over finite fields. The plus operation is similar to the operation we defined previously for elliptic curve over real numbers.

The identity element is same as before. The reverse pair of elements are symmectircal over the line $$y = P/2$$.

For any pointers $$m$$ and $$n$$ on the discrete curve and $$x_m \ne x_n$$, $$m \oplus n$$ is the reverse proint of the 3rd intersection between the discrete curve points and the lines denoted by:

$$ax + by + c\equiv 0 \mod P$$

when one of the lines cross $$m$$ and $$n$$, and $$a$$, $$c$$ and $$c$$ are inetger constants.

The following is an example, where the discrete elliptic curve equation is $$y^2 \equiv x^3 - 7x + 10 \mod 19$$, and the lines equation is $$y \equiv 2x \mod 19$$:

![](http://tao93.top/images/2020/03/29/1585518712.jpeg)

In above diagram, $$m$$ and $$n$$ are $$(1, 2)$$ and $$(5, 10)$$ respectively, while $$m \oplus n$$ is $$(17, 4)$$. So this is a general example, how about when $$m$$ is just $$n$$? What's the $$\circledcirc$$ is this group?

The $$\circledcirc$$ is still the infinitely far point. Well, this seems strange becase the infinitely far point is not in the discrete elliptic curve. And there has $$m \oplus n = \circledcirc$$ when $$x_m = x_n, y_m \ne y_n$$.

When $$m$$ is just $$n$$, we can also get the tangent line whose slope value is the derivative of the elliptic curve equation at the point $$m$$.

It could be proved with such a plus operation defined, a group is generated on the discrete elliptic curve over a finite field from prime $$P$$.

The number of points (including the $$\circledcirc$$) in such a group is called the **order** of the group. The [Schoof's algorithm](https://en.wikipedia.org/wiki/Schoof%27s_algorithm) is an efficient algorithm to calculate the number of points.

Similar with previous continuous elliptic curve, we can perform scalar multipilication:

$$dm = \sum_{i=1}^d m$$

where $$d$$ is a positive integer and $$m$$ is an element in the group. 

For any $$m$$ in such a group $$G$$, the smallest integer $$d$$ such that $$dm$$ equals $$\circledcirc$$ is called the order of the subgroup which consists of $$\circledcirc$$, $$m$$, $$2m$$, $$3m$$, ..., $$(d-1)m$$. The subgroup is cyclic. And the number of elements in a subgroup is called **order** of the subgroup.

Let's say the order of the parent group is $$D$$. According to [Lagrange's theorem](https://en.wikipedia.org/wiki/Lagrange%27s_theorem_(group_theory)), the order of a subgroup is a diviser of $$D$$. 

As an example, lets use the $$y^2 \equiv x^3 - 7x + 10 \mod 19$$ and assum3 $$m$$ is $$(1, 2)$$. It could be calculated that $$m, 2m, 3m, 4m, 5m, 6m, 7m, 8m$$ are $$(1, 2), (18, 15), (9, 12), (7, 0), (9, 7), (18, 4), (1, 17), \circledcirc$$. Show as below:

![](http://tao93.top/images/2020/03/29/1585522273.jpeg)

In above diagram, the order of the parent group is 24. The order of the subgroup is 8, which is a diviser of 24. Also for the point $$(7, 0)$$, its inverse point is itself (actually the slope value here is infinity).

Note that different subgroups might share points. For instance, in the above example any subgroup whose base point is $$m$$ order is an even number $$s$$, then $$\frac{s}{2}m$$ is the point $$(7, 0)$$.

Hence for any $$m$$ in the parent group, we can check the divisers of $$D$$ one by one to find out the order of subgroup based on $$m$$. 

On the contrary, for any prime diviser $$d$$ of $$D$$, any element point $$m$$ such that $$(D/d)m$$ not equal the $$\circledcirc$$ is the base point to form a subgroup whose order is $$d$$. (Hint: $$Dm$$ is always the $$\circledcirc$$.)

Given $$m$$ and $$n$$ in such a group G, getting a $$k$$ such that $$km=n$$ is also very hard for some instances of discrete elliptic curve, just like the logrithm problem we mentioned previously.

#### Elliptic curve cryptography applications

To leverage elliptic curve group over finite fileds on primes, there are some parameters called domain parameters to determine the concrete elliptic curve instances.

1. $$a$$ and $$b$$, real numbers to determine the $$y^2 = x^3 + ax + b$$ equation.
2. $$p$$, a prime that determine the finite field of 0, 1, 2, 3, ..., $$p-1$$ .
3. $$G$$, a base point in the discrete elliptic curve to form a cyclic subgroup of $$G$$, $$2G$$, $$3G$$, ..., $$\circledcirc$$.
4. $$n$$, which is order of the subgroup based on $$G$$, hence $$nG$$ is the $$\circledcirc$$.
5. $$h$$, called cofactor, which is result of $$n$$ divides the order of the parent group.

The above $$(p, a, b, G, n, h)$$ consists the domain parameters that determine the concrete instances. Thoso domain parameters are public to everyone.

In the Elliptic curve cryptography, The **private key** is a random number $$d$$ such that $$ 1 \le d \le n-1$$. The **public key** is the result of $$dG$$.

##### EC Diffie-Hellman key exchange algorithm

In EC Diffie-Hellman key exchange algorithm. Alice and Bob have their private keys $$d_A$$ and $$d_B$$. With the public domain parameters, they would compute $$d_A G$$ and $$d_B G$$ and exchange these two values (public keys). Finally, they would compute $$d_A d_B G$$ and $$d_B d_A G$$ respectively as the secret key for symmetric encryption. Of course $$d_A d_B G = d_B d_A G$$.

The EC Diffie-Hellman key exchange is based on that given the domain parameters, it's hard to know $$d_A d_B G$$ given $$d_A G$$ and $$d_B G$$, and same for $$d_A$$ and $$d_B$$.

The shared secret key in EC Diffie-Hellman key exchange algorithm are coordinates of the point $$d_A d_B G$$, which could be used to perform symmectirc encryption just like TLS does.

Actually in TLS, we often see ECDHE in which the ending E means ephemeral which means the shared secret key is temporary. Actually when TLS uses ECDHE, during the handshake procedure, the public and private keys generated in ECDHE won't be used for encryption. TLS just want both sides to generate a shared private symmectric encryption key, which is really for encryption of application data.