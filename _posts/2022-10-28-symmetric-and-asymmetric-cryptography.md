---
layout: post
title: "Algorithms in Symmetric and Asymmetric Cryptography"
excerpt: "A Brief Introduction to AES and RSA"
---

Cryptographic schemes can be categorized into two major types: Symmetric and Asymmetric. In modern day cryptography, these two types of cryptographic schemes are used to ensure confidentiality, authentication, and integrity of our internet traffic. Without them, every message we send through the internet would be sent in plaintext, allowing malicious users to listen in and eavesdrop on the conversation.

Here I will go over one algorithm from both categories: AES for symmetric cryptography and RSA for asymmetric cryptography. They both have their strengths and weaknesses, and their use cases in the real world vary because of these differences.

&nbsp;
## AES (Advanced Encryption Standard)

AES stands for Advanced Encryption Standard. It is essentially a block cipher, which encrypts text one block at a time. It is currently the most recommended and widely used symmetric cryptographic algorithm. As with all symmetric algorithms, AES uses one key that is used to both encrypt the plaintext and decrypt the ciphertext. Block sizes are typically 128 bits (16 bytes), and key sizes vary from 128 bits (16 bytes) to 256 bits (32 bytes).

AES and symmetric algorithms in general are considered faster than their asymmetric counterparts, which is why they are preferred for encrypting data transferred over the internet. Compared to RSA, which I will cover below, AES can encrypt large amounts of data more efficiently and requires a shorter key length to ensure confidentiality. However, it is not without its disadvantages.

Having a single key for both encryption and decryption provides a single point of failure; if the key is compromised, then the confidentiality of the message is compromised. Given the purpose of cryptographic schemes and symmetric encryption, the sender of a message and the recipient must both have knowledge of the key. There is no way for the two parties to securely share this key since a symmetric encryption algorithm always requires a single key for secure transmission. This means that if you choose to use only symmetric encryption, then the only way for you to safely exchange the secret key with your intended recipient is to physically write down the key on a piece of paper and deliver the note in person, which completely undermines its ultimate goal of secure online data transfer.

Another problem with AES is that it is a block cipher, which means the same block of plaintext will always be encrypted to the same ciphertext when the same key is used. When encrypting a single block, it is a deterministic algorithm. The security problems caused by this characteristic can be best demonstrated using the encrypted image of the famous Linux penguin:

<p align="center">
	<img src="/assets/images/Penguins/ecb_penguin.png" />
</p>

The above image, as you can see, is an encrypted image of the Linux penguin. However, despite being encrypted with AES, the silhouette of the penguin is very clearly visible; the encryption did no good in ensuring secrecy of the message. However, it should be noted that AES comes with various “modes of encryption” to mitigate this vulnerability. The above image was encrypted using AES in ECB mode, which encrypts each plaintext block individually and just appends the ciphertext blocks together to form the final ciphertext. As showcased above, this is not secure and definitely not recommended. More secure modes of encryption for AES exist: CBC and CFB to name a few. Let’s see the same Linux penguin encrypted using AES in CBC mode:

<p align="center">
	<img src="/assets/images/Penguins/cbc_penguin.png" />
</p>

As you can see, the penguin is completely unrecognizable. CBC stands for Cipher Block Chaining mode. As its name suggests, it is a mode of AES encryption in which every plaintext is xor-ed with the previous ciphertext block before being encrypted with the key. This ensures that even if the single plaintext block is identical to another plaintext block, it will be encrypted to a different ciphertext block due to it being xor-ed with the previous ciphertext block. When using AES to encrypt data, it is crucial to understand the different modes of encryption since its default mode of encryption is ECB which is not secure.

&nbsp;
## RSA (Rivest, Shamir, Adleman)

RSA is one of the most well known asymmetric cryptographic schemes that exist today. Its name takes after the three scientists at MIT that came up with the algorithm. Unlike AES, RSA is an asymmetric cryptographic algorithm that uses more than one key to encrypt and decrypt the message. Its key consists of one public key and one private key. In the RSA system, everyone has a public key that can be distributed to anyone that wishes to send an encrypted message to the owner of that public key. Then once the message is encrypted and transmitted online to the recipient, only the correct recipient that has the private key matching the public key can decrypt the message successfully.

RSA keys are typically 1024 bits (128 bytes) to 4096 bits (512 bytes). Compare this to AES’s maximum key size of 256 bits (32 bytes) and you can see why AES is preferred when encrypting large amounts of data. RSA tends to be slower and less efficient than other symmetric algorithms like AES. However, despite needing more computational resources for encryption and decryption, RSA has a few advantages over AES.

First of all, RSA is very secure. Even with its public key being exposed, it is mathematically and realistically impossible to break RSA encryption as long as the private key is kept secret. Moreover, its use of public key cryptography enables it to cover other areas of security that cannot be replaced by symmetric encryption algorithms. To elaborate, the public-private key system allows its users to ensure not only confidentiality but also integrity, non-repudiation, and authenticity of data.

A real world example of how useful RSA can be is better introduced when compared to the problems of AES mentioned above. Previously I said that AES struggles with the initial key exchange, since both the sender and the recipient need to know the secret key. Well, RSA does not have that problem; the parties involved simply need to share their public keys with each other. It does not matter if a malicious attacker can eavesdrop and retrieve the public key, since this information is not enough for the encryption to be broken. In the real world, asymmetric algorithms like RSA are used for the initial key exchange. The key being exchanged is usually a symmetric key for an encryption scheme like AES. Once this initial connection is established and both parties have knowledge of the secret AES key, they can securely continue the conversation with the much faster AES algorithm.

Another use for asymmetric algorithms like RSA is message authentication and digital signatures. Having digital signatures on a message provides additional security in that the recipient can be confident in the message's integrity and authenticity. If the sender of a message creates a digital signature by encrypting a known piece of text with their private key, then the recipients can decrypt that digital signature with the sender’s public key. Since only the legitimate sender has access to their private key, the recipient can rest assured that the message was not spoofed by a malicious attacker.

Similarly, RSA provides integrity in the sense that an attacker tampering with the content of the message without knowledge of the private key will essentially break the encryption. RSA relies heavily on the mathematical correctness of the values of the ciphertext, the public key, and the private key. If the ciphertext is somehow tampered with, then the recipient trying to decrypt it with their private key will only receive a broken plaintext message, which notifies them of potential compromise of message integrity.

&nbsp;
## Conclusion

These are the most widely used encryption algorithms for symmetric and asymmetric cryptography. Both have their advantages and disadvantages, and these characteristics make them fit for use in different real world circumstances. As software engineers that aim to develop secure software, we should all be familiar with these industry standard cryptographic schemes and learn to identify improper use of these algorithms which may compromise your system to malicious hackers.
