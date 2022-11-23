---
layout: post
title: "ARP Cache Poisoning"
excerpt: "Attacks Against ARP"
date: "2022-11-16"
---

For my Network Security course we are assigned a number of labs relevant to network security throughout the semester. Some examples of the topics covered in these hands-on labs are:

- Packet capture and analysis using WireShark
- Scanning test IPs using NMAP
- Creating snort rules to simulate an Intrusion Detection System

While these labs were certainly interesting, I recently finished a lab involving ARP attacks which I found to be most interesting - enough for me to write about it here.

&nbsp;
## So... what *is* ARP?

ARP stands for "Address Resolution Protocol". As its name suggests, ARP is a protocol used to resolve a machine's physical MAC address to its IP address. One problem with ARP is that it was never designed with security in mind. Which means malicious attackers can mess with the ARP cache to trick a host with incorrect MAC & IP pairs. This is what we were trying to simulate in this lab.

&nbsp;
## Setting up the lab environment

The lab was done through [Immersive Labs](https://www.immersivelabs.com/){:target="_blank"}.
The environment was set up using Docker containers to simulate three machines: two victim machines (A and B) and one malicious attacker machine (M).

<p align="center">
	<img src="/assets/images/Arp_Attack/setup.png" />
</p>

&nbsp;
## ARP cache poisoning

My first task was to use an attack known as ARP cache poisoning. To do this I used a packet manipulation tool called Scapy, which allows me to mess with packets in python.

The code below shows a simple script to send a fake ARP request to machine A which will "poison" its ARP cache to map the MAC address of machine M to the IP address of machine B. The `psrc` and `hwsrc` fields are set accordingly when crafting this fake ARP request. Take note of the IP and MAC addresses of all three machines, as we will be referencing these values to confirm that the attack worked when checking the victim's ARP cache.

<p align="center">
	<img src="/assets/images/Arp_Attack/attack_code1.png" />
</p>

After running the code, we can `docksh` into our machine A. If we check its ARP cache, we can see that it has been "poisoned" with the wrong mapping of machine M's MAC address to machine B's IP address. Now, if there is IP traffic intended for machine B, it will be directed to machine M according to machine A's ARP cache.

<p align="center">
	<img src="/assets/images/Arp_Attack/arp_cache1.png" />
</p>

&nbsp;
## MITM using ARP cache poisoning

The next step in the lab was to implement a Man-In-The-Middle (MITM) attack over Netcat using ARP cache poisoning. The general idea of this attack is to trick both machines A and B into thinking that their IP addresses are mapped to machine M's MAC address. All messages that A sends to B and vice versa will go through machine M, and the attacker in control of machine M can eavesdrop or even modify the packets before forwarding them to its recipient.

First step is to poison the ARP cache of both A and B using the following code:

<p align="center">
	<img src="/assets/images/Arp_Attack/attack_code2.png" />
</p>

We can see that the ARP cache of both A and B points to the MAC address of machine M.

ARP cache of machine A:

<p align="center">
	<img src="/assets/images/Arp_Attack/arp_cacheA.png" />
</p>

ARP cache of machine B:

<p align="center">
	<img src="/assets/images/Arp_Attack/arp_cacheB.png" />
</p>

After enabling IP forwarding on machine M, we can intercept and modify packet contents between A and B. For demonstration purposes, I created a script to replace all instances of my name "Kang In" with "AAAA AA".

The script:

<p align="center">
	<img src="/assets/images/Arp_Attack/mitm_code.png" />
</p>

While I have the script running on machine M, I established a Netcat connection between A and B and started sending messages from A to B. We can see that whenever A sends my name "Kang In", B only sees "AAAA AA". Cool!

<p align="center">
	<img src="/assets/images/Arp_Attack/mitm_demo.png" />
</p>

&nbsp;
## Conclusion

I had previously heard about ARP poisoning attacks and MITM attacks, but this was the first time I implemented it first hand. It was a really interesting lab that showcased how malicious attackers can intercept and inject false messages into a victim's packet contents. If this sounds interesting, I encourage you to check out Immersive Labs!