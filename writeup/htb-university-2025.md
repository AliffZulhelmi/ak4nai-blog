---
description: Forensics only
---

# HTB UNIVERSITY 2025

## Santa Giveaway \[MEDIUM]

**Description:**\
At Wintercrest Workshop, an employee ran a cheerful holiday giveaway helper that left behind a shimmerdust‑thin trail no one noticed at first. That single action set off a quiet compromise beneath the system’s surface. You are provided only with a full memory dump. Reconstruct the incident using volatile artifacts: identify the process that began the chain, uncover the in‑memory traces it left behind, and extract the command line showing how the intruder secured its foothold

**Solution:**

{% hint style="info" %}
**.vmem** file is a copy of everything that was inside a Virtual Machine' RAM at a specific moment
{% endhint %}

Based on given memory image (vmem) file, I'm using **volatility3** to solve this challenge.

{% stepper %}
{% step %}
During initial compromise, What is the URL that the victim downloaded the malware from, to start the compromise? (for example: https://example.com)

**http://graveyard.htb:8000**
{% endstep %}

{% step %}
What date(time) did the user initiated the download? (UTC Time: YYYY-MM-DD hh:mm:ss)

**2025-11-22 15:07:23**
{% endstep %}

{% step %}
Which process became the primary malicious executable? Provide the PID associated with this process. (for example: 1337)

6520
{% endstep %}

{% step %}
Based on memory analysis, what is the location the malware was originally launched from? Please, provide the full path.

**C:\Users\user\AppData\Local\Temp\152c6d54a1\rgbux.exe**
{% endstep %}

{% step %}
When reviewing network activity, which remote IP address and port did the malicious process communicate with? (IP:PORT)

**89.58.51.107:80**
{% endstep %}

{% step %}
What is the Malware Family

Amadey
{% endstep %}

{% step %}
What is the GUID of the scheduled task created by the malware (HKLM...)

**81A3950E-EE73-4DB9-B670-DF3979056B48**
{% endstep %}

{% step %}
As part of maintaining a long‑term foothold, what full registry key did the malware modified to ensure it launches at every user session startup? (HKCU...)

**HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Startup**
{% endstep %}
{% endstepper %}

<details>

<summary>POC</summary>

URL visited by user often keep inside browser local db. I begin my investigation by finding the browser used by the victim.

```
vol.py -r csv -f Challenge-Snapshot.vmem windows.netscan > netscan.csv
```

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Based on the evidence above, the victim used Chrome to access the URL. As for chrome, its history stored in sqlite db located at `C:\Users<username>\AppData\Local\Google\Chrome\User Data\Default`&#x20;

Then, I scan files that present inside the memory image, we can utilize grep to find the history file in hundreds of output listed.

```
vol.py -r csv -f Challenge-Snapshot.vmem windows.filescan > files.csv
```

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Now, I acknowledged that **history db** is present inside the memory image.  We can dump this database using it's virtual addr on the left.

```
vol.py -f Challenge-Snapshot.vmem windows.dumpfiles --virtaddr 0xe786d47c1ef0
```

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

To ensure that we get a the right file, I confirm the file type from the file header

```
xxd file.0xe786d47c1ef0.0xe786d37d7010.DataSectionObject.History.dat | head
```

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Now, we can begin our investigation on our next artifact (history.db)

```
cp file.0xe786d47c1ef0.0xe786d37d7010.DataSectionObject.History.dat history.db
sqlite3 history.db
.tables
```

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

The db file seems malformed, but we can recover this using sqlite recover feature.

```
sqlite3 history.db ".recover" > history_recovered.sql
sqlite3 history_recovered.db < history_recovered.sql
sqlite3 history_recovered.db
```

Once the database file recovered. We begin our investigation on the artifact. I begin by identify available tables inside the db and print the table content. A table named "urls" have caught my interest.

```
sqlite3 history_recovered.db
SELECT * FROM urls;
```

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

I see the suspicious link accessed by the victim. Which is **http://graveyard.htb:8000.** Now let's identify the timestamp. when the victim download the malicious file.

```
SELECT * FROM downloads;
```

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Based on **downloads** table, the victim installed a **malicious** application "named" DiscordGiveaway.exe at **13408297643715783.** Chrome history db used a webkit timestamp format to store date n time information. we can convert this value using online converter and get **2025-11-22 15:07:23.**&#x20;

Let's begin our investigation post malware installation. we can identify malicious application running using **malfind.**

```
vol.py -f Challenge-Snapshot.vmem windows.malware.malfind
```

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Based on **malfind** output, **rgbux.exe** is a malicious application run with pid **6520**. Using the pid, we can scan files running, and use the file name to look up on **Virustotal**.

```
vol.py -f Challenge-Snapshot.vmem windows.filescan | grep "rgbux"
vol.py -f Challenge-Snapshot.vmem windows.dumpfiles --pid 6520 --virtaddr 0xe786d5170d70
```

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Based on files available in the memory, there's to entry, **0xe786d5170d70** which is executable file inside the **C:\Users\user\AppData\Local\Temp\152c6d54a1\rgbux.exe**, meanwhile **0xe786d51f3ba0** being cached. I dumped the files on **0xe786d51f3ba0** virtual address, and I got 2 files, one end with **.exe.dat** and another end with **.exe.img.** Our primary target is **.img** file, we can hash the file and lookup on **Virustotal.**

```
sha256sum file.0xe786d5170d70.0xe786d5b435d0.ImageSectionObject.rgbux.exe.img
```

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

From the lookup on virustotal, This malware is part of **Amadey** malware family. Let's head back to volatility to identify any **connection** made by this malware.

```
vol.py -r csv -f Challenge-Snapshot.vmem windows.netscan.NetScan > netscan.csv
```

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Based on the netscan result, We can conclude the malware made a connection to **89.58.51.107:80** possibly a C2 Server. Now, I moved on to finding persistence technique used by the malware. Based on the virustotal report, the malware create a schedule task (T1053). We can confirm this via **window.registry.scheduled\_tasks.**

```
vol.py -f Challenge-Snapshot.vmem windows.registry.scheduled_tasks | grep "rgbux.exe"
```

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Based on the output, we can confirm the malware does created a scheduled task with GUID **81A3950E-EE73-4DB9-B670-DF3979056B48.** Not only that, the report also mentioned the malware modify a registry key to keep persistence access. We can investigate it using **strings.**&#x20;

```
strings Challenge-Snapshot.vmem | grep "Shell Folders"
```

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

Based on both analysis, The malware created a process that add persistence key to "**HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Startup**". I have a high confident, the malware perform similar behavior on this victim too.

</details>
