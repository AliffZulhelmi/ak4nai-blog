---
description: >-
  Here are the write-ups for blue team challenges that I solved during the
  competition
---

# The Card \[Easy]

## Overview

**Description:**

Holmes receives a breadcrumb from Dr. Nicole Vale - fragments from a string of cyber incidents across Cogwork-1. Each lead ends the same way: a digital calling card signed JM.

## Solution

Players required to submit 12 flags to pwn the machine

### **Flag 1**

**Question:** Analyze the provided logs and identify what is the first User-Agent used by the attacker against Nicole Vale's honeypot. (string)

**Answer:** Lilnunc/4A4D - SpecterEye

**Explanation:** Based on analysis on **access.log** file given. The first suspicious request made by custom user-agent on robots.txt.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### **Flag 2**

**Question:** It appears the threat actor deployed a web shell after bypassing the WAF. What is the file name? (filename.ext)

**Answer:** temp\_4A4D.php

**Explanation:** Based on **waf.log,** A rule "WEBSHELL\_DEPLOYMENT" are triggered. The action describe that a **PHP web shell "temp\_4A4D.php" created.**

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Flag 3

**Question:** The threat actor also managed to exfiltrate some data. What is the name of the database that was exfiltrated? (filename.ext)

**Answer:** database\_dump\_4A4D.sql

**Explanation:** Based on **waf.log,** A rule "DATABASE\_DOWNLOAD" triggered. The action describe the attacker have downloaded database file "database\_dump\_4A4D.sql" which might store critical data.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### Flag 4

**Question:** During the attack, a seemingly meaningless string seems to be recurring. Which one is it? (string)

**Answer: 4A4D**

**Explanation:** I have encountered string "4A4D" multiple times used on name of user-agent, web shell, and dumped file thru out the analysis.

### Flag 5

**Question:** OmniYard-3 (formerly Scotland Yard) has granted you access to its CTI platform. Browse to the first IP:port address and count how many campaigns appear to be linked to the honeypot attack.

**Answer: 5**

**Explanation:** Based on **CogWork-Intel Graph** platform, there's multiple entities of different category linked to one or another. Using information I got on **honeypot logs** I reviewed, the attacker matched organization "**JM**" profile. This threat actor linked to **5 different campaign.**

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>



### Flag 6

**Question:** How many tools and malware in total are linked to the previously identified campaigns? (number)

**Answer: 9**

**Explanation:** I calculated number of **tools and malware (**&#x73;uch as **MedSys Probe)** linked to **these 5 campaigns (**&#x73;uch as **QuantumCoin Stealer)**.

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

### **Flag 7**

**Question:** It appears that the threat actor has always used the same malware in their campaigns. What is its SHA-256 hash? (sha-256 hash)

**Answer:** 7477c4f5e6d7c8b9a0f1e2d3c4b5a6f7e8d9c0b1a2f3e4d5c6b7a8f9e0d17477

**Explanation:** I inspected all linked malware and identify entities "IoC" that indicates these different malware. All these malware share the same pattern and assigned with SHA-256 hash which is the same on all malware on different campaigns.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

### Flag 8

**Question:** Browse to the second IP:port address and use the CogWork Security Platform to look for the hash and locate the IP address to which the malware connects. (Credentials: nvale/CogworkBurning!)

**Answer:** 74.77.74.77

**Explanation:** I open the link given, which is an **Advanced Threat Analysis Platform "CogWork Security"** which have a **HASH Lookup** features, I utilize hash that I found to find where's the malware connected to. Based on the result, the malware connected to a **C2 Infra with IP Addr "74.77.74.77".**

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### Flag 9

**Question:** What is the full path of the file that the malware created to ensure its persistence on systems? (/path/filename.ext)

**Answer:** /opt/lilnunc/implant/4a4d\_persistence.sh

**Explanation:** The detailed analysis on the malware includes behavioral analysis. Behavioral analysis section indicate the malware created a bash file "4a4d\_persistance.sh". The file might contain a script used to establish persistence connection between C2 Infra and victim machine.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

### Flag 10

**Question:** Finally, browse to the third IP:port address and use the CogNet Scanner Platform to discover additional details about the TA's infrastructure. How many open ports does the server have?

**Answer:** 11

**Explanation:** I open the 3rd link, it's a "CogNet Scanner". I searched for C2 Infra (74.77.74.77) information, and there's multiple information provided including **Overview, Services, Vulnerabilities, and etc.** Under **Services** section, there's a list of services and port opened.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

### Flag 11

**Question:** Which organization does the previously identified IP belong to? (string)

**Answer:** SenseShield MSP

**Explanation:** Under **Overview** section, **Network Information** also provide the name of organization that IP belongs to.

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### Flag 12

**Question:** One of the exposed services displays a banner containing a cryptic message. What is it? (string)

**Answer:** He's a ghost I carry, not to haunt me, but to hold me together - NULLINC REVENGE

**Explanation:** I'm scrolling through services that's open under **Services** section and encounter an unclear message at **7477/tcp port** which can be a cryptic message.

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
