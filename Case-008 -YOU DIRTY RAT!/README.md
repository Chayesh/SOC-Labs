# 2024-07-30 - Traffic Analysis Exercise: YOU DIRTY RAT!

## Overview

This investigation analyzes a packet capture (PCAP) from a Windows enterprise environment. The objective was to identify the infected host, determine the affected user, and investigate the malicious network activity observed after the compromise.

---

# Executive Summary

On **30 July 2024 at approximately 02:40 UTC**, a Windows workstation within the `wiresharkworkshop.online` domain exhibited malicious network activity consistent with **STRRAT (Strigoi Remote Access Trojan)**.

During the investigation, the infected host established encrypted connections to several external services including GitHub infrastructure and Maven repositories before contacting an external STRRAT command-and-control (C2) server.

The malware periodically transmitted beacon messages containing host information, operating system details, Windows Defender status, user activity, and active window titles.

No malware sample was recoverable from the available packet capture. The PCAP contains post-infection network activity rather than the initial malware delivery.

---

# Victim Details

| Field | Value |
|--------|-------|
| Host Name | DESKTOP-SKBR25F |
| IP Address | 172.16.1.66 |
| MAC Address | 00:1e:64:ec:f3:08 |
| Windows User | ccollier |
| Full Name | Clark Collier |
| Domain | wiresharkworkshop.online |

---

# Investigation Process

## 1. Victim Identification

The infected workstation was identified through network traffic analysis and LDAP authentication records.

Victim details were confirmed using:

- Kerberos
- LDAP
- SMB traffic

---

## 2. Suspicious HTTPS Traffic

The victim communicated with several external services before the malware began beaconing.

Observed domains:

- github.com
- objects.githubusercontent.com
- repo1.maven.org

These connections are consistent with software or payload retrieval and required further investigation.

---

## 3. Public IP Discovery

The infected host queried:

GET /json/

Host:

ip-api.com

This request retrieved the victim's public IP address and geolocation.

---

## 4. STRRAT Command and Control Activity

Subsequent TCP traffic revealed repeated STRRAT heartbeat messages.

Example:

ping|STRRAT|1BE8292C|DESKTOP-SKBR25F|ccollier|Microsoft Windows 11 Pro|64-bit|Windows Defender|...

The beacon contained:

- Victim ID
- Hostname
- Username
- Operating System
- Architecture
- Antivirus
- Country
- Idle status
- Active window title

This confirms successful communication with the STRRAT command-and-control server.

---

## 5. Base64 Encoded Telemetry

Several Base64-encoded strings were observed within the beacon traffic.

Decoded values included:

| Encoded | Decoded |
|----------|----------|
| SG9tZQ== | Home |
| RG9jdW1lbnRz | Documents |
| UGljdHVyZXM= | Pictures |
| UHJvZ3JhbSBNYW5hZ2Vy | Program Manager |

These values represent active folders or application window titles monitored by STRRAT.

---

# Indicators of Compromise (IOCs)

## Victim

- Hostname: DESKTOP-SKBR25F
- User: ccollier
- Full Name: Clark Collier
- IP Address: 172.16.1.66
- MAC Address: 00:1e:64:ec:f3:08

---

## External Domains

- github.com
- objects.githubusercontent.com
- repo1.maven.org
- ip-api.com

---

## Command & Control

IP Address:

141.98.10.69

Port:

12132/TCP

Malware:

STRRAT (Strigoi Remote Access Trojan)

---

# Attack Timeline

02:39 UTC

- Victim workstation operating normally.

↓

02:39–02:40 UTC

- HTTPS connections established to GitHub infrastructure and Maven repository.

↓

02:40 UTC

- Public IP address retrieved using ip-api.com.

↓

Immediately afterward

- STRRAT established communication with the external C2 server.

↓

Post Infection

- Periodic beacon messages transmitted every few seconds.
- Malware reported:
  - Hostname
  - Username
  - Operating System
  - Windows Defender status
  - Active window titles
  - Idle status

---

# Conclusion

The packet capture provides clear evidence that the Windows workstation **DESKTOP-SKBR25F** was infected with **STRRAT**.

The malware established persistent communication with its command-and-control server while continuously reporting host information and user activity.

Although the initial infection vector is not present within the packet capture, the observed network traffic confirms a successful post-compromise STRRAT infection.

---

# Skills Practiced

- Wireshark packet analysis
- Victim identification
- LDAP analysis
- SMB analysis
- TLS/SNI investigation
- Command-and-Control detection
- STRRAT traffic analysis
- Base64 decoding
- IOC extraction
- Incident reporting