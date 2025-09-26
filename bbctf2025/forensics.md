---
description: >-
  Here are the write-ups for forensic challenges that I solved within the
  competition or after competition.
icon: file-magnifying-glass
---

# Forensics

### Challenge 1 - F\*\*\* Microsoft

#### Overview

**Description**:&#x20;

OMG. I HATE WINDOWS UPDATE. HOW CAN YOU AUTO UPDATE WHEN I'M 80% DONE WITH MY ASSIGNMENT. I DIDN’T SAVE ANYTHING, IT BETTER AUTO RECOVERS.

{% file src="../.gitbook/assets/dist.7z" %}
Compressed challenge
{% endfile %}

This .7z file contains _F Microsoft.ad1.txt_ and _F Microsoft.ad1._

> Info: F Microsoft.ad1.txt is a metadata file that the ftk image software generates.&#x20;
>
> Meanwhile, F Microsoft.ad1 is an actual image file that we will use.

#### 1. Import .ad1 into FTK Imager

I'm using Exterro FTK Imager, there are other alternatives like AccessData FTK Imager.

_How to import the .ad1 file_? Click File > Add Evidence Item > Image File > Locate your .ad1 file

#### 2. Locate the assignment file

Normally, people place their work files in directories that are easy to access. I look up at the common directories such as Desktop, Downloads, Documents and OneDrive.&#x20;

We found it inside _\~/Desktop/Assignment_

<figure><img src="../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

In the challenge's description, it highlighted about the assignment file is not saved. So the file that we found at _\~/Desktop/Assignment_ is probably an incomplete file.

#### 3. Locate Auto-Save MS Word Location

After a quick research, I found the location of MS Word auto recovery file location.

> Windows:
>
> **Default Location:** The default location for AutoRecover files is `C:\Users\<Your_username>\AppData\Roaming\Microsoft\Word`

<figure><img src="../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

I located the file using previous information, so we found it. Now let's move it to our machine that has MS Word. I pasted the file in the same directory path on my machine to make sure .asd file is compatible with MS Word.

> Note: .asd is an auto recovery file format used by MS Word.

<figure><img src="../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

### Challenge 2 - Notes

#### Overview

**Description:** I wrote the flag in notepad and accidentally closed it. Will I lose the flag?

#### 1. Import .ad1 Into FTK Imager

Click File > Add Evidence Item > Image File > Locate your .ad1 file

#### 2. Locate auto-save / cache location

Based on challenge's description, the challenge hint us to locate the location of auto-save / cache of Notepad file. At first, I thought it was memory analysis but after a bit of more research i find out that it's not the intended way of solving this challenge.

Utilizing my google fu, I discovered the cache location for Notepad.

> The Notepad artifacts are located at the path `%LOCALAPPDATA%\Packages\Microsoft.WindowsNotepad _8wekyb3d8bbwe\LocalState`, this directory contains two folders:
>
> **TabState** - This folder contains files for each tab in Notepad. These files contain the most interesting data.
>
> **WindowsState** - This folder contains information about the Notepad windows (e.g., the currently focused tab, etc.).

From this information, I located the notepad binary file.

<figure><img src="../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>
