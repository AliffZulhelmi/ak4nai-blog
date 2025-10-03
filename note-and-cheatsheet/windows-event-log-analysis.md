---
description: Note and Cheat sheet
---

# Windows Event Log Analysis

## Event Log Introduction

### Purpose

The event logs record events that happen on the computer. Examining the events in these logs can help you trace activity in forensics process.

### Basics of Windows Event Logs

* **Stored** in `Windows\System32\winevt\logs`&#x20;
* **Format** for the logs is `.evtx`
* **ADDITIONAL |** Event logs can be collected via **Event Viewer**, **wevtutil** or forwarded to **SIEM.**
* Main Logs are categorized to:
  * **Security** - Account / Logon Events
  * **System** - Service Startup / Shutdown
  * **Application** - App related issues
  * **Application** and Services Logs - Detailed service specific logs

### What's inside the logs

Event IDs have several fields in common:

<details>

<summary><strong>Important fields in logs</strong></summary>

<table><thead><tr><th width="161">Field</th><th>Description</th></tr></thead><tbody><tr><td>Event ID</td><td>A code assigned to each type of audited activity.</td></tr><tr><td>Level</td><td>The severity of the recorded event.</td></tr><tr><td>User</td><td>User account involved in the activity. </td></tr><tr><td>Computer</td><td>The host which the event was logged.</td></tr><tr><td>Source</td><td>The service, component or application that generated the event.</td></tr><tr><td>Description</td><td>A description on the recorded event, where additional information specific to the event being logged. <strong>Most significant field for analyst</strong></td></tr></tbody></table>

</details>

***

## Forensics Tools

* **EvtxECmd** - Parsing Windows Event Logs
* **Event Viewer  -** View Windows Event Logs

***

## Windows Event Logs Cheat sheet

### Account Management Events

Useful to track account creation, modification, and deletion.

<details>

<summary>Useful Event ID and Description</summary>

<table><thead><tr><th width="151">Event ID</th><th>Description</th></tr></thead><tbody><tr><td><strong>4720</strong></td><td>User account created</td></tr><tr><td>4722 / 4725</td><td>Account enabled / disabled</td></tr><tr><td>4723 / 4724</td><td>Password change or reset attempt</td></tr><tr><td><strong>4726</strong></td><td>Account deleted</td></tr><tr><td>4732 / 4733</td><td>User added/removed from group</td></tr><tr><td>4741 / 4743</td><td>Computer account created/deleted</td></tr><tr><td>4798 / 4799</td><td>Account or group enumeration [<strong>FLAGGED FOR RECON ACTIVITY</strong>]</td></tr></tbody></table>

</details>

### Logon & Authentication Events

> _**Account Logon** is term for authentication, meanwhile **Logon** refers to account that gaining access to a resources._

**Account Logon (Authentication)** is the act of verifying user's credential. Both information events will be recorded in the **Security** event log.

* Authentication of domain accounts is performed by a domain controller.
* Authentication of local accounts is performed by the local system.

<details>

<summary>Useful Event ID and Description</summary>

**Domain Controllers (Authentication)**

<table><thead><tr><th width="130">Event ID</th><th>Description</th></tr></thead><tbody><tr><td><strong>4768</strong></td><td><p>Kerberos TGT issued </p><p>(success/failure codes reveal lockouts, bad passwords, expired tickets).</p></td></tr><tr><td>4769 / 4770</td><td>Service ticket requested/renewed.</td></tr><tr><td><strong>4771</strong></td><td>Failed Kerberos logon.</td></tr><tr><td><strong>4776</strong></td><td><p>NTLM authentication attempt.  </p><p>Look for multiple failures <strong>(POSSIBLE PASSWORD GUESSING)</strong>.</p></td></tr></tbody></table>

**On the accessed system (Logon/Logoff)**

<table><thead><tr><th width="130">Event ID</th><th>Description</th></tr></thead><tbody><tr><td><strong>4624</strong></td><td>Successful logon. (Interactive, Network, Remote Desktop)</td></tr><tr><td><strong>4625</strong></td><td>Failed logon. Status codes explain why (bad password, expired, lockout).</td></tr><tr><td>4634 / 4647</td><td>User logged off.</td></tr><tr><td><strong>4648</strong></td><td>Logon using explicit credentials (RunAs, UAC bypass).</td></tr><tr><td><strong>4672</strong></td><td>Logon with admin privileges.</td></tr><tr><td>4778 / 4779</td><td>RDP session reconnected/disconnected.</td></tr></tbody></table>

</details>

### Access to Shared Objects Event

Attackers frequently leverage valid credentials hunt for sensitive files via network shares

<details>

<summary>Useful Event ID and Description</summary>

<table><thead><tr><th width="130">Event ID</th><th>Description</th></tr></thead><tbody><tr><td>5140</td><td>Network share accessed.</td></tr><tr><td>5145</td><td>File access checked (enables detailed share auditing).</td></tr><tr><td>5142–5144</td><td>Share created/modified/deleted.</td></tr></tbody></table>

</details>

### Scheduled Tasking Logging

Attackers frequently utilize task scheduler to ensure persistence access.

<details>

<summary>Useful Event ID and Description</summary>

<table><thead><tr><th width="130">Event ID</th><th>Description</th></tr></thead><tbody><tr><td>106 / 140 / 141</td><td>Task created, updated, deleted.</td></tr><tr><td>200 / 201</td><td>Task executed / completed.</td></tr><tr><td>4698 / 4702</td><td>Task created/updated (with full XML details).</td></tr></tbody></table>

</details>

### Object Access Auditing

Attacker sometimes modify sensitive files, make changes in folders or on registry.&#x20;

This log tracks sensitive file/folder/registry access.

<details>

<summary>Useful Event ID and Description</summary>

<table><thead><tr><th width="130">Event ID</th><th>Description</th></tr></thead><tbody><tr><td>4656</td><td>Handle requested for object (who tried to access).</td></tr><tr><td><strong>4663</strong></td><td>Object access attempt (read/write/modify).</td></tr><tr><td><strong>4657</strong></td><td>Registry value modified.</td></tr><tr><td><strong>4660</strong></td><td>Object deleted.</td></tr><tr><td>4663</td><td>File copied to removable storage (USB/External Drive)</td></tr></tbody></table>

</details>

### Windows Services

Services are often abused by attacker to ensure persistence access.

<details>

<summary>Useful Event ID and Description</summary>

<table><thead><tr><th width="130">Event ID</th><th>Description</th></tr></thead><tbody><tr><td>6005 / 6006</td><td>Event log service started/stopped.</td></tr><tr><td><strong>7036</strong></td><td>Service started/stopped.</td></tr><tr><td><strong>7040</strong></td><td>Service start type changed.</td></tr><tr><td><strong>7045</strong></td><td>Service installed (watch for random names, or executables in Temp).</td></tr><tr><td><strong>4697</strong></td><td>Service installed (with Advanced Audit Policy enabled).</td></tr></tbody></table>

</details>

### Wireless LAN Auditing

Logs wireless Lan activity

<details>

<summary>Useful Event ID and Description</summary>

<table><thead><tr><th width="130">Event ID</th><th>Descriptiopn</th></tr></thead><tbody><tr><td><strong>8001</strong></td><td>Connected to Wi-Fi (SSID, profile, auth type).</td></tr><tr><td><strong>8002</strong></td><td>Failed Wi-Fi connection attempt.</td></tr></tbody></table>

</details>

### Process Tracking & Command Monitoring

Attacker can craft an attack using Living-off-the-Land (LotL) which the attacks doesn't involve creating on modifying files. It's critical log for spotting this attack.

<details>

<summary>Useful Event ID and Description</summary>

<table><thead><tr><th width="167">Event ID</th><th>Description</th></tr></thead><tbody><tr><td>4688</td><td>New process created (can include command line if enabled).</td></tr><tr><td>5031 / 5152–5159</td><td><p>Windows Filtering Platform events </p><p>(network activity blocked/allowed).</p></td></tr></tbody></table>

</details>

### Auditing PowerShell Use

PowerShell = attacker’s best partner-in-crime

<details>

<summary>Useful Event ID and Description</summary>

<table><thead><tr><th width="130"></th><th></th></tr></thead><tbody><tr><td><strong>4103</strong></td><td>Pipeline execution (module logging).</td></tr><tr><td><strong>4104</strong></td><td>Script block logging (captures commands).</td></tr><tr><td>400 / 800</td><td>Execution/session start, with encoded commands detection.</td></tr></tbody></table>

</details>
