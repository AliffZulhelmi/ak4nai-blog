---
description: HTB Machine writeup
icon: linux
---

# Conversor

## Overview

<figure><img src="../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

## Reconnaissance

### Port Scanning

I started recon by performing port scanning. I usually start with rustscan for faster port discovery and move to nmap for detailed scanning.

```
rustscan -a 10.10.11.92
```

<figure><img src="../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

```
nmap -p 22,80 10.10.11.92
```

<figure><img src="../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

### Port 80 | HTTP

Login page is served as a landing page. Nothing much interest on tech stack based on wappalyzer.

<figure><img src="../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

I then move on to register page, checking out on tech stack using wappalyzer. Same result, nothing interest. I created an account to login.

<figure><img src="../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

Once I logged in, I served with a file upload section, an XML and XSLT input file.

<figure><img src="../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

## Scanning

### Source Code Analysis

### XSLT Injection Enumeration

I retrieved vender and version information using the payload

```
// XML | test.xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <data>Exploit</data>
</root>

// XSLT | test.xslt
<?xml version="1.0" encoding="UTF-8"?>
<html xsl:version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl">
<body>
<br />Version: <xsl:value-of select="system-property('xsl:version')" />
<br />Vendor: <xsl:value-of select="system-property('xsl:vendor')" />
<br />Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" />
</body>
</html>
```

**Finding**

<figure><img src="../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

