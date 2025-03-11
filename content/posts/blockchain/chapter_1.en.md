---
layout: post
title: "Understand All Common Security Algorithms in One Article"
description: "Introduce all common security algorithms, including symmetric encryption, asymmetric encryption, digital signatures, etc."
date: 2025-03-09T14:54:27+08:00
image: "/posts/blockchain/images/chapter_1-cover.jpg"
tags: ["Blockchain"]
---
## Information Security Issues

When data is exchanged over the network, it passes through various network environments and devices. During the transmission process, it may pass through the devices of some malicious users, which could lead to the theft of the content. When transmitting data over the network, four main issues are usually considered.

* Eavesdropping: Messages are eavesdropped on during transmission.
* Impersonation: Falsely claiming to be the sender or recipient of a message.
* Tampering: The message was tampered with during transmission.
* Denial: The act of a message sender denying their own act of sending the message afterwards.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-main-problems.jpg" width="100%">
<p>Main Problems</p>
</div>

To solve the above problems, we need to use some security algorithms. Let's take a look at how these algorithms address these issues.

## Security Algorithm

### Encryption Algorithm

To address the issue of eavesdropping, we need to have the sender of the message encrypt it with an encryption key, and then have the recipient decrypt the ciphertext with a decryption key upon receipt to obtain the actual content of the message. Since what is actually transmitted is the ciphertext, even if it is eavesdropped on during transmission, the eavesdropper cannot obtain the actual content of the message without the decryption key. There are two algorithms for encrypting and decrypting messages: one is "symmetric encryption", where the same key is used for both encryption and decryption, and the other is "asymmetric encryption", where different keys are used for encryption and decryption.

#### Symmetric Encryption

Symmetric encryption involves the sender using a key to encrypt the message and transmitting the ciphertext to the recipient, who then decrypts the ciphertext with the same key to obtain the original message. Even if the ciphertext is intercepted by a third party, without the key, the original message cannot be obtained. Common symmetric encryption algorithms include AES and DES.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-symmetric-encryption.jpg" width="80%">
<p>Symmetric Encryption</p>
</div>

However, if the sender and the receiver cannot communicate directly and the receiver does not know the ciphertext at the beginning and needs the sender to send the ciphertext to the receiver, then the key is also at risk of being eavesdropped on, and at this point, the security of the message cannot be guaranteed. Therefore, it is necessary to find a way to send the key securely, which is the "key distribution problem". To solve the key distribution problem, two methods can be used: a secure "key exchange protocol" (such as the Diffie-Hellman key exchange) or "asymmetric encryption".

#### Asymmetric Encryption

Asymmetric encryption can solve the "key distribution problem" that exists in symmetric encryption. The message recipient prepares a pair of public and private keys, with the public key being publicly released and the private key kept by themselves. When the message sender is ready to send a message, they encrypt the message using the recipient's publicly available public key and transmit the ciphertext to the recipient. The recipient then decrypts the ciphertext using their private key to obtain the message. Common asymmetric encryption algorithms include RSA and elliptic curve encryption.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-asymmetric-encryption.jpg" width="80%">
<p>Asymmetric Encryption</p>
</div>

However, suppose a third party secretly replaces the public key of the message recipient with its own public key. The message sender will then encrypt the message using the third party's public key. At this point, the third party can eavesdrop on the ciphertext, decrypt it with its own private key to obtain the message, and then re-encrypt the message with the recipient's public key to generate a new ciphertext and send it to the recipient. The recipient will not be aware that the message has been eavesdropped on. This attack method of eavesdropping on messages by replacing public keys is called a "man-in-the-middle attack". To solve the reliability problem of public keys, "digital certificates" need to be used.

In addition, the encryption and decryption operations of asymmetric encryption algorithms are relatively time-consuming, so this method is not suitable for the continuous sending of fragmented messages. To avoid the "key distribution problem" and ensure efficiency, a "hybrid encryption" algorithm needs to be used.

#### Hybrid Encryption

Symmetric encryption has the problem of key distribution, which makes it impossible to securely transmit the key. Asymmetric encryption, on the other hand, has the issue of relatively slow encryption and decryption speeds. A hybrid encryption method that combines these two approaches to complement each other is a solution. The core idea is to use the shared key of symmetric encryption to encrypt and decrypt messages, and to use asymmetric encryption to securely transmit the shared key.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-hybrid-encryption.jpg" width="60%">
<p>Hybrid Encryption</p>
</div>

Hybrid encryption has advantages in both security and processing speed. The SSL/TLS protocol, which provides communication security for networks, also applies the hybrid encryption method. Of course, hybrid encryption cannot avoid the problem of impersonation caused by the inability to guarantee the source of the public key.

### Verification Algorithm

To address the issues of impersonation, tampering and denial, we need to verify the identity of the message sender and the content of the message.

#### Message Authentication Code

A message authentication code is an algorithm that can verify whether a message has been tampered with during transmission. In addition to exchanging encrypted messages, the communicating parties also exchange message authentication codes. The sender and the receiver share a key for generating the message authentication code. The sender uses this key to calculate the message authentication code from the encrypted message and sends both the encrypted message and the message authentication code to the receiver. After receiving the message, the receiver generates an authentication code using the same key and compares it with the received authentication code. If they are the same, it indicates that the encrypted message has not been tampered with.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-message-verification-code.jpg" width="80%">
<p>Message Verification Code</p>
</div>

Of course, if the message recipient and sender cannot communicate directly, sharing the message authentication code generation key will encounter the "key distribution problem". At the same time, since both communication parties hold the authentication code generation key, if one party denies sending the message after sending it and falsely accuses the other party of sending it, the message authentication code cannot identify it. "Digital signature" can solve these two problems very well.

#### Digital Signature

Digital signatures not only can achieve the function of detecting tampering like message authentication codes, but also can prevent the problem of post-denial, because a digital signature can only be generated by the sender.

To use digital signatures, the sender first needs to prepare a pair of public and private keys, which is contrary to asymmetric encryption where the receiver prepares the public and private keys. When the sender sends a message, they encrypt the hash value of the message with their private key to generate a digital signature and send the signature along with the message to the receiver. After receiving the message and the signature, the receiver performs a hash operation on the message to obtain the hash value and decrypts the digital signature with the sender's public key to obtain the hash value of the received message. By comparing the two hash values, if they are equal, it indicates that the message comes from the sender and has not been tampered with.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-digital-signature.jpg" width="50%">
<p>Digital Signature</p>
</div>

But just like asymmetric encryption and hybrid encryption, digital signatures cannot guarantee the source of the public key, so they cannot solve the problem of impersonation. To address this issue, "digital certificates" need to be used.

### Digital Certificate

Digital certificates are used to prove the source information of public keys. They are issued by trusted third-party institutions, which are called Certification Authorities (CAs). When someone wants to publish a public key and make it trustworthy, they need to apply for a digital certificate for that public key from a CA. First, they need to submit their information, email address, and the public key they want to publish to the CA. After verifying the information and email address, the CA issues a digital certificate for the public key, which is essentially a digital signature of the applicant's information and public key using the CA's private key.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-digital-certificate.jpg" width="50%">
<p>Digital Certificate</p>
</div>

When others obtain this digital certificate, they verify its credibility through the public key of the certification center. After the verification is successful, the source of the public key is confirmed. In this way, the risk of the public key being replaced is avoided.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-certification-authority.jpg" width="50%">
<p>Certification Authority</p>
</div>

You might think that the public key of the certification authority could also be replaced. How can we ensure the credibility of the public key of the certification authority? In fact, the public key of the certification authority is also in the form of a digital certificate, usually issued by a higher-level certification authority, forming a tree structure. The topmost certification authority in this structure is called the "Root CA", which proves its own legitimacy. If the root certification authority is not trusted, the entire organization cannot operate. Therefore, root certification authorities are mostly large enterprises or government certification organizations.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_1-ca-tree.jpg" width="60%">
<p>CA Tree</p>
</div>

Digital certificates are like this, guaranteeing the creator of public keys through certification authorities. This series of technical specifications is collectively referred to as "Public Key Infrastructure" (PKI).

## Summary

* When transmitting data over the network, four major security issues need to be considered: eavesdropping, impersonation, tampering and denial.

* Encryption algorithms can solve the problem of eavesdropping. Such algorithms include symmetric encryption, asymmetric encryption, and hybrid encryption that combines the two. Hybrid encryption not only ensures the security of the shared key but also guarantees the efficiency of encryption.

* The verification algorithm can solve the problems of tampering and denial. Among them, the message authentication code can solve the problem of tampering, while the digital signature can solve both the problems of tampering and denial.

* For asymmetric encryption, hybrid encryption and digital signature algorithms that use asymmetric keys, all need to deal with the problem of impersonation caused by the replacement of public keys, and digital certificates can ensure the credibility of the authenticated public keys.