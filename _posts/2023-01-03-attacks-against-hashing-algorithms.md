---
layout: post
title: "Attacks Against Hashing Algorithms"
excerpt: "Hash Collisions and Rainbow Tables"
date: "2023-01-03"
---

In a world where data is an invaluable asset, we all know the importance of properly understanding and implementing cryptographic systems to ensure confidentiality of data. When it comes to confidentiality, we often discuss symmetric and asymmetric cryptographic algorithms such as AES and RSA. However, today I want to talk about another aspect of cryptography: hashing algorithms.

Hashing algorithms are a crucial aspect of cryptography in any modern application including but not limited to secure messaging apps, online file sharing, and even blockchain technology. Throughout the history of modern cryptography, there have been a number of different hashing algorithms. Some are more secure than others, and some have become obsolete due to security reasons. There are different types of vulnerabilities and attacks against some of these hashing algorithms, and I believe that any software engineer working closely with secure data should have a rudimentary understanding of these issues and how hashing algorithms have evolved to overcome them.


&nbsp;
## What is a Hashing Algorithm?

There are several important properties of a hashing algorithm that make it cryptographically viable.

First, a hashing algorithm must be deterministic. If the same input is provided to a hashing function, its output should always be the same. Second, a hash function must be a one-way function. In other words, it should realistically be impossible to figure out the input when given only the output. Third, a hashing algorithm must always produce an output of fixed length regardless of the input length. This property allows hashing algorithms to reduce large size inputs into consistent length outputs. 

Given these properties, it may be tempting to ask why hashing algorithms are needed in the first place. If it is deterministic and cannot be reversed, how useful can it be in securing sensitive data? After all, there are many cryptographic algorithms like RSA and AES that can already provide adequate data confidentiality. To understand the benefits of using hashing algorithms, let’s examine a few of its use cases.


&nbsp;
### Use Case 1: Securing Password Database

Before hashing algorithms became standard practice in the industry, it wasn’t uncommon for databases containing sensitive data to store them in unencrypted plaintext. The world in general lacked a cybersecurity mindset, and it was only after a few major database breach cases that companies started to prioritize a multi-layered approach to securing their sensitive data. While storing user passwords in the database as plaintext wasn’t an issue by itself, the problem was that the confidentiality of those user passwords solely relied on the security of the database. If the database was compromised, every information stored in that database was compromised; the database was its single point of failure.

A famous example of these password breaches is Rockyou. The Rockyou password list is a 100 GB text file containing more than 8 billion entries of the most common passwords, all extracted from past data breaches targeting databases that stored passwords in plaintext. The list first started circulating in an online hacker forum, and is often used as a tool for executing or demonstrating password dictionary attacks. 

To prevent these breaches from happening again, a hashing algorithm can be used to hash the password before storing them in the database. The idea is simple - when a user registers an account with a password, the server will put it through the hash function and store the hash value in the database instead of the plaintext value. This way, when an attacker gains unauthorized access to the database, they still won’t be able to derive the actual passwords from the password hash since a hash function is irreversible. Verifying passwords during login is also an easy task. When a user types in their password, the server will receive the password, put it through the same hash function, and compare the hash value with the one stored in the database. Recall that a hash function is deterministic - meaning that the same password input will always give the same output, allowing the server to verify the correct password.

<p align="center">
	<img src="/assets/images/Hashing/password_hashing.png"
		 width=500 />
</p>


&nbsp;
## Use Case 2: Online File Sharing

While the internet has made it really easy to share files and download documents, one of the major problems with internet security is that these files can often be malicious, containing malware that will sabotage the host computer without the user even realizing it. Attackers can execute Man-In-The-Middle attacks to intercept file transfers and modify its contents to something fake or even dangerous. On some websites that provide download files, they often provide a “checksum” value next to the file. Here is an example from the download page of Arch Linux:

<p align="center">
    <img src="/assets/images/Hashing/arch_linux_checksum.png"
         width=700 />
</p>

So, what are these values? SHA256 and BLAKE2b are two different hashing algorithms that are being used to verify the integrity of the download file. Since a file is, at its core, just a sequence of bytes, the file can be put through a hashing algorithm as the input. The hash value seen in the screenshot is the hash output of that particular file. If an end user wanted to make sure that the file is not being modified in transit by an outside attacker, the user would first download the file, and then run the file contents through the specified hashing algorithm (in this case, SHA256 or BLAKE2b) and compare the hash value with the one listed on the website. If the two values are different, it means that the local file is different from the one listed on the website, which could indicate a potential compromise to file integrity. 


&nbsp;
## Use Case 3: Blockchain

Hashing algorithms are also an essential part of blockchain networks - it is a core concept of Bitcoin’s consensus algorithm (Proof-of-Work) and provides an efficient way to verify that a new block has been mined properly with the correct transaction data. As we have discussed in our course, a blockchain network is collectively maintained by everyone in the network. To put it simply, a blockchain network is a distributed ledger / database in which every node in the network keeps track of every transaction that has happened thus far. However, as one can imagine, having everyone in the network keep a record of all past transactions would be highly inefficient, especially as time goes on and the history of transactions becomes larger in size. To solve this problem, blockchain networks use hashing algorithms to reduce the size of these records. Recall that an important property of hashing algorithms is that it produces the same length output regardless of the input size. By just keeping track of the hash value of the previous block on the blockchain, the network can efficiently reference this hash value to run the Proof-of-Work consensus algorithm and create a new hash value which verifies all past transaction records until the current block.

<p align="center">
    <img src="/assets/images/Hashing/bitcoin_hash.png"
         width=500 />
</p>


&nbsp;
## Attacks Against Hashing Algorithms

Like the word “encryption”, “hashing algorithm” is a broad term that can refer to a variety of hashing algorithms. There are many well known hashing algorithms that have become the industry standard, but before these modern algorithms came to be, there existed older hashing algorithms that are now obsolete due to disclosed vulnerabilities.

### Collision Attack

One attack against hashing algorithms is called the “collision attack”. As its name suggests, it is when two different inputs result in the same hash output, resulting in a “collision”. Let’s take the hash password database as an example to see the implications of a collision attack. First, a user creates their account with a password. However, it turns out that the specific password phrase they chose is susceptible to a hash collision - meaning there exists another alphanumeric phrase that returns the same hash output. In theory, this means that an attacker, or any outsider, can log into the account without using the correct passphrase. Instead, they can use the specific phrase that causes the hash collision, and log into the victim’s account. While this example is not realistically probable, there are other applications for these attacks - these hash collisions can be exploited to forge online certificates or digital signatures. Ultimately, having easy-to-find hash collisions is not an ideal property of a secure hash algorithm.

However, it is inevitable that a hash function has collisions. To understand why, we can refer to a concept called the Pigeonhole Principle. The principle states that given X number of pigeonholes and Y number of pigeons, and there are more pigeons than pigeonholes, then there must be at least one pigeonhole that contains more than one pigeon. The image below demonstrates this concept eloquently:

<p align="center">
    <img src="/assets/images/Hashing/pigeonhole.png"
         width=200 />
</p>

So how is this relevant to hashing algorithms? Well, in the case of hashing functions, the input size is the number of pigeons, and the output size is the number of pigeonholes. Recall that all hashing algorithms produce the same size output regardless of input size. The input size of a hashing algorithm is unlimited, whereas its output is always of fixed size (128 bits in the case of MD5). Because of this, it is inevitable that there exists an output that maps to more than one input, resulting in a hash collision.

<p align="center">
    <img src="/assets/images/Hashing/pigeonhole_function.png"
         width=200 />
</p>

### Rainbow Tables

Another attack against vulnerable hash algorithms is something called a “Rainbow Table”. It is also referred to as the preimage attack. This attack abuses the fact that all hashing algorithms are deterministic - an input will always map to the same output. It is essentially a pre-computed brute-force attack against a particular hashing algorithm. A rainbow table refers to a huge database mapping all possible inputs to its hashed output. Given these details, older hashing algorithms are naturally more susceptible to rainbow table attacks than more modern ones.

Some rainbow tables for obsolete hashing algorithms are available online, and anyone can use it to do a reverse look-up search to find the original input based on the hashed output. Crackstation is one of those online tools that provide reverse look-up search for hashing algorithms (mainly for MD5 and SHA1). According to their website, they use a 190 GB database with 15 billion entries to perform the reverse search. As one can imagine, building a rainbow table from scratch requires a lot of computing power and time. However, once these tables are built and disclosed online, the targeted hashing algorithm becomes vulnerable to rainbow table attacks.

<p align="center">
    <img src="/assets/images/Hashing/rainbow_table.png"
         width=400 />
</p>


&nbsp;
## Obsolete Hashing Algorithms

Let’s examine a few obsolete hashing algorithms and try to understand why they are not recommended for use in modern cryptography.

### MD5

MD5 is the third iteration of a family of hashing algorithms also known as “Message Digest”. The MD5 hashing algorithm produces a 128-bit hash value, which is one of the smallest output sizes for hashing algorithms. It was first developed in 1991 by Ronald Rivest, who also contributed to creating the RSA algorithm. In fact, the “R” in RSA stands for his name, Rivest. While MD5 is still widely used today, it is usually only used for quickly checking file checksums or data integrity. It is not recommended to use MD5 to hash sensitive data, as it has already been shown to be cryptographically broken and susceptible to collision attacks and rainbow table attacks.

### SHA1

While a little more secure than MD5, SHA1 is another example of a cryptographically insecure hashing algorithm. It produces a 160-bit hash value, which is slightly better than MD5, but still not good enough. It was developed by the NSA in 1995, but has been considered cryptographically broken since 2005. Similar to MD5, it is prone to hash collision attacks as well as rainbow table attacks, and it should not be used in applications dealing with sensitive data.


&nbsp;
## Secure Hashing Algorithms

While the hashing algorithms above have been known to be broken for a long time in the cryptography community, many developers today still use these hashing functions without fully understanding the risks of doing so. According to a 2017 research conducted by Venafi Labs, 21% of websites were still using the SHA1 hashing algorithm. That is one in every five websites that is vulnerable to attacks targeting SHA1. So what hashing algorithm should developers and software engineers use in building a secure application?

### SHA256

The SHA256 is one of many hashing functions that reside in the family of SHA2 hashing algorithms. It is the hashing function of choice for Bitcoin, and is the recommended industry standard for most modern applications today. As its name reveals, it produces a 256-bit hash value. It was developed by the NSA in 2001 along with many other flavors of hashing algorithms in the SHA2 family. Currently, it has remained a secure hashing algorithm to be used in modern software engineering - no collisions have been found, and its output size is big enough to deter any meaningful attempts at building a rainbow table. If you are building a modern application that deals with sensitive data, SHA256 should be the first that comes to mind.


&nbsp;
## Conclusion

While many hashing algorithms have been proven to be insecure for years, not every developer is well educated in which ones they should opt for when starting a new project. I believe that all software engineers should be familiar with the recommended standards for using these cryptographic algorithms, and hopefully the industry will see more secure applications properly using the correct hashing algorithms for their intended purpose.

