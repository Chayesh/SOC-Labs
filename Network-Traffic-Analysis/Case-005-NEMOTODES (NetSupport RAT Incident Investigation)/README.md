# Case 005 - NEMOTODES (NetSupport RAT Incident Investigation)

> **Platform:** Malware-Traffic-Analysis.net  
> **Exercise Date:** 2024-11-26  
> **Difficulty:** Intermediate–Advanced  
> **Category:** Network Traffic Analysis / Incident Response / Threat Intelligence

---

# Objective

Investigate a packet capture (PCAP) and IDS alerts to identify an infected Windows workstation, reconstruct the attack timeline, identify malicious infrastructure, and document all Indicators of Compromise (IOCs).

Unlike previous exercises, this investigation required correlating packet analysis with IDS alerts and external threat intelligence to understand the complete infection chain.

---

# Scenario

A Security Operations Center (SOC) for a medical research facility detected multiple IDS alerts indicating malicious activity within the network.

The investigation focused on:

- Identifying the infected Windows workstation
- Determining the associated Windows user
- Validating IDS alerts through packet analysis
- Identifying malware infrastructure
- Producing an incident report with supporting evidence

---

# Environment

| Item | Value |
|------|-------|
| LAN Range | 10.11.26.0/24 |
| Domain | nemotodes.health |
| Active Directory | NEMOTODES |
| Domain Controller | NEMOTODES-DC (10.11.26.3) |
| Gateway | 10.11.26.1 |

---

# Initial Investigation

Unlike earlier exercises, the investigation began with reviewing the provided IDS alerts before opening the PCAP.

This approach mirrors a real SOC workflow, where analysts typically pivot from alerts into packet analysis.

---

# Alert Review

Primary observations from the IDS alerts:

- Multiple DNS lookups for **modandcrackedapk.com**
- TLS connections associated with the same domain
- NetSupport RAT Command-and-Control alerts
- HTTP POST requests over TCP port 443
- NetSupport geolocation lookup activity
- Repeated beaconing traffic

The primary suspicious internal host was identified as:

```
10.11.26.183
```

---

# Investigation Process

## Step 1 - Identify the Victim

Reviewing SMB and Windows authentication traffic identified the infected workstation.

| Item | Value |
|------|-------|
| Victim IP | 10.11.26.183 |
| MAC Address | d0:57:7b:ce:fc:8b |
| Hostname | DESKTOP-B8TQK49 |

---

## Step 2 - Establish Initial Timeline

The beginning of the capture showed only legitimate Windows startup behavior.

Observed activity:

- Microsoft Internet Connectivity Test
- LDAP communication
- WPAD discovery
- Internal DNS traffic

No malicious activity was observed during the initial packets.

This established a baseline of normal Windows network behavior before the suspicious activity began.

---

## Step 3 - Suspicious DNS Activity

The first notable malicious indicator appeared as DNS lookups for:

```
modandcrackedapk.com
```

This represented the first observed suspicious network activity within the capture.

---

## Step 4 - TLS Communication

Shortly after the DNS resolution, the workstation established TLS communication associated with the same infrastructure.

This indicated successful communication with external malicious infrastructure.

---

## Step 5 - NetSupport RAT Command-and-Control

Filtering traffic for:

```wireshark
ip.addr == 194.180.191.64
```

revealed repeated HTTP POST requests over TCP port 443.

HTTP stream analysis showed:

```http
POST http://194.180.191.64/fakeurl.htm HTTP/1.1

User-Agent: NetSupport Manager/1.3

Host: 194.180.191.64
```

This communication pattern matched the IDS alerts indicating NetSupport RAT Command-and-Control activity.

---

## Step 6 - Threat Intelligence Correlation

The suspicious domain:

```
modandcrackedapk.com
```

was investigated using external threat intelligence sources.

ThreatFox identified the domain as part of a **FakeUpdate (SocGholish / SmartApeSG)** malware campaign.

Observed campaign chain:

```
Compromised Website
        ↓
Injected JavaScript
        ↓
Fake Browser Update
        ↓
ZIP Payload
        ↓
NetSupport RAT
        ↓
194.180.191.64
```

Although the complete delivery chain was documented through threat intelligence, the JavaScript and ZIP payload were **not recoverable from the available packet capture**.

---

# Attack Timeline

| Time (UTC) | Event |
|------------|-------|
| 04:49:38 | Normal Windows startup networking (Microsoft Connectivity Test, LDAP, WPAD) |
| 04:50:14 | DNS lookup for modandcrackedapk.com |
| 04:50:40 | TLS connection to malicious infrastructure |
| 04:50:45 | HTTP POST over TCP port 443 |
| 04:50:45 | NetSupport RAT C2 communication begins |
| 04:50:45 | Repeated beaconing to C2 server |
| 04:50:45 | Geolocation lookup activity |

---

# Evidence Correlation

The compromised workstation was validated using multiple independent evidence sources.

### IDS Alerts

- DNS lookup alerts
- NetSupport RAT alerts
- HTTP POST over port 443
- Geolocation lookup alerts

---

### Network Traffic

- DNS queries
- TLS communication
- HTTP POST requests
- NetSupport Manager User-Agent

---

### Windows Network Activity

- SMB
- LDAP
- NetBIOS
- Hostname identification

---

### Threat Intelligence

ThreatFox classified:

```
modandcrackedapk.com
```

as infrastructure associated with the **FakeUpdate (SocGholish / SmartApeSG)** malware campaign.

---

# Indicators of Compromise (IOCs)

## Victim

| Item | Value |
|------|-------|
| IP Address | 10.11.26.183 |
| MAC Address | d0:57:7b:ce:fc:8b |
| Hostname | DESKTOP-B8TQK49 |

---

## Domains

```
modandcrackedapk.com
```

---

## IP Addresses

```
194.180.191.64
104.26.1.231
193.42.38.139
```

---

## URLs

```
http://194.180.191.64/fakeurl.htm
```

---

## User-Agent

```
NetSupport Manager/1.3
```

---

## Malware

```
NetSupport RAT
```

Campaign Attribution:

```
FakeUpdate
SocGholish
SmartApeSG
```

---

# Malware Extraction

The exercise requested SHA256 hashes if malware binaries could be extracted from the packet capture.

During analysis:

- HTTP Export Objects were reviewed.
- No executable, DLL, ZIP, or other malware binaries were recoverable from the provided PCAP.
- Threat intelligence indicates the campaign typically delivers a ZIP payload; however, this payload was not present within the captured traffic.

Therefore, no SHA256 hash could be calculated from the available evidence.

---

# Detection Techniques Used

- IDS Alert Analysis
- Timeline Reconstruction
- DNS Analysis
- TLS Analysis
- HTTP Stream Analysis
- SMB Analysis
- NetBIOS Analysis
- Threat Intelligence Correlation
- IOC Pivoting
- Evidence Correlation

---

# Executive Summary

A Windows workstation (DESKTOP-B8TQK49) on the internal network communicated with infrastructure associated with the FakeUpdate (SocGholish / SmartApeSG) malware campaign. Following DNS resolution of **modandcrackedapk.com**, the host established outbound communication and initiated repeated HTTP POST requests over TCP port 443 using the **NetSupport Manager/1.3** User-Agent. Packet analysis validated IDS alerts indicating NetSupport RAT command-and-control activity with **194.180.191.64**. Although threat intelligence links this campaign to malicious JavaScript and ZIP payload delivery, those payloads were not recoverable from the provided packet capture.

---

# Lessons Learned

- Begin investigations with IDS alerts before analyzing packets.
- Establish a baseline of normal network activity before identifying malicious behavior.
- Validate IDS detections using packet analysis instead of relying solely on alert descriptions.
- Correlate network evidence with external threat intelligence to understand the broader malware campaign.
- Recognize that packet captures may not contain every stage of an attack; the initial infection vector or malware payload may fall outside the capture window.

---

# Skills Practiced

- Wireshark
- Network Forensics
- Incident Response
- IDS Alert Validation
- Timeline Reconstruction
- Threat Intelligence
- NetSupport RAT Analysis
- DNS Analysis
- HTTP Analysis
- TLS Analysis
- IOC Extraction
- Evidence Correlation