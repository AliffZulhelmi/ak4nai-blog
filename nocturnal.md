---
description: Here are the write-ups for HTB machine, Nocturnal.
icon: linux
---

# Nocturnal

This machine initially provides an IP Address only.

## Scanning

### Port Scanning

Scanning for any active ports and services running. Utilizing rustscan for quick scanning and nmap for extensive information of services running. Outcome of these scans hint us that port 22,80 and 9999 is open.

#### Rustscan Port Scanning

```
rustscan -a 10.10.11.64
```

<figure><img src=".gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

#### Nmap Port Scanning

```
nmap -sC -sV -p- 10.10.11.64
```

<figure><img src=".gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

Now we got the domain name for the ip, let's include it inside `/etc/hosts`&#x20;

```
10.10.11.64    nocturnal.htb
```

### Port 22 Enumeration0

I found a few CVEs associate with OpenSSH 8.2p1. But, nothing worked out here for me.

### Port 80 Enumeration



```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://nocturnal.htb/FUZZ -fc 302,301
```

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt -u http://nocturnal.htb/FUZZ -fc 302,301
```

<figure><img src=".gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>
