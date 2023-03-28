---
layout: post
title: "OSINT, SIEMS, and Data Processing Part 1"
excerpt: "Identifying threats and attacks"
date: "2023-03-04"
---

In one of the labs for my Security Analytics class, we learned to use Splunk, Pyspark, and the VirusTotal API to identify threats from large amounts of data. In modern day threat intelligence, manually sifting through the huge amount of data for Indicators of Compromise (IOC) is nearly impossible. SIEM tools like Splunk allow us to manage data at scale, while tools like Apache Spark help us process the big data into useful information. Services like VirusTotal allow us to verify whether a given URL, IP address, or file hash is an Indicator of Compromise, implying it may be linked to malware.


&nbsp;
## The Task

My first task for the lab was to analyze a given PCAP file and determine when the malware infection happened and what the malware was. The PCAP file was fairly large, and just manually sifting through packets with WireShark was going to be unrealistic. Therefore, I fed the PCAP data into Splunk, which gave me a better high level overview of the packets.

<p align="center">
    <img src="/assets/images/OSINT_SIEMS/splunk_graph.png"
         width=600 />
</p>


&nbsp;
## DNS Observations

From the Splunk graph above, I could tell that network traffic dramatically increased at 9:52PM, suggesting possible infection / malware activity. Based on the DNS analysis, there were countless DNS requests made to resolve the following URL: `am[.]super1024[.]com`. Using VirusTotal, I could tell that this was a malicious URL and possibly the infection source for the malware. Based on further research, the URL is related the "Adylkuzz" malware, which I will discuss more below.

<p align="center">
    <img src="/assets/images/OSINT_SIEMS/virustotal_super1024.png"
         width=900 />
</p>


&nbsp;
## HTTP Breakdown

Based on the HTTP analysis, I noticed a suspiciously large number of requests sent to one specific IP address. This address is suspected to be the malicious URL mentioned above. The infected victim computer was making HTTP requests to the malicious URL, most likely due to the malware.

HTTP requests to malicious endpoint:
<p align="center">
    <img src="/assets/images/OSINT_SIEMS/splunk_destination_ip.png"
         width=500 />
</p>

HTTP requests from victim host:
<p align="center">
    <img src="/assets/images/OSINT_SIEMS/splunk_source_ip.png"
         width=500 />
</p>


&nbsp;
## Malware Steps

On further analyzing the HTTP packets, we can see multiple HTTP GET requests made to the malicious endpoint.

<p align="center">
    <img src="/assets/images/OSINT_SIEMS/splunk_http.png"
         width=500 />
</p>

Based on my research of the "Adylkuzz" malware, these GET requests serve different roles.
- The initial malware install starts with `GET /86.exe`.
- To report that the malware was installed, `GET /report` and `GET /install` are used.
- I believe that empty `GET /` requests are made to check if the malware needs any software updates.
- Most importantly, the `GET /mine.txt` is used to download cryptomining instructions and configuration files - hence the name `mine.txt`.


&nbsp;
## Summary

Based on the PCAP file, it seems that the system was infected with a cryptocurrency miner known as "Adylkuzz". The malware consumes computer resources to mine "Monero", a cryptocurrency. The attack started at 9:52PM. There is a sudden increase in the number of packets transmitted starting at this time - these packets attempt to contact URLs related to the "Adylkuzz" malware. The victim IP that was infected seems to be `192.168.1.114`