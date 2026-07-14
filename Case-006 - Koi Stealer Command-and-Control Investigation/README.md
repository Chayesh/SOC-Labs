# Case 006 - Koi Stealer Command-and-Control Investigation

> **Platform:** Malware-Traffic-Analysis.net  
> **Exercise Date:** 2024-09-04  
> **Difficulty:** Intermediate  
> **Category:** Incident Response / Malware Traffic Analysis / Command-and-Control Analysis

---

# Objective

Investigate a packet capture (PCAP) and IDS alerts to identify an infected Windows workstation, validate Koi Stealer command-and-control (C2) activity, and document the associated Indicators of Compromise (IOCs).

---

# Scenario

A Security Operations Center (SOC) received IDS alerts indicating malware activity within the corporate network.

The investigation focused on:

- Identifying the infected Windows workstation
- Validating IDS alerts using packet analysis
- Identifying command-and-control infrastructure
- Producing an incident report with supporting evidence

---

# Environment

| Item | Value |
|------|-------|
| LAN Range | 172.17.0.0/24 |
| Domain | bepositive.com |
| Active Directory | BEPOSITIVE |
| Domain Controller | WIN-CTL9XBQ9Y19 (172.17.0.17) |
| Gateway | 172.17.0.1 |

---

# Initial Investigation

The investigation began by reviewing the provided IDS alerts before examining the packet capture.

The alerts indicated repeated communication between an internal workstation and an external server associated with Koi Stealer command-and-control activity.

Primary suspicious host:

```
172.17.0.99
```

---

# Investigation Process

## Step 1 - Identify the Victim

Windows authentication and SMB traffic identified the compromised workstation.

| Item | Value |
|------|-------|
| Victim IP | 172.17.0.99 |
| MAC Address | 18:3d:a2:b6:8d:c4 |
| Hostname | DESKTOP-RNVO9AT |
| Windows User | afletcher |
| Full Name | Andrew Fletcher |

---

## Step 2 - Validate IDS Alerts

The IDS generated repeated alerts indicating Win32/Koi Stealer command-and-control activity.

Packet analysis confirmed repeated communication between:

```
172.17.0.99
        ↓
79.124.78.197
```

This validated the IDS detections.

---

## Step 3 - Analyze HTTP Communication

Following the HTTP stream revealed multiple requests associated with the command-and-control session.

Observed requests:

```http
POST /foots.php
```

```http
GET /index.php?id=&subid=qIOuKk7U
```

```http
POST /index.php
```

The communication used:

```
Host:
79.124.78.197
```

and included the User-Agent:

```
Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; WOW64; Trident/7.0 ...)
```

---

## Step 4 - C2 Communication

The traffic pattern suggests a typical malware command-and-control workflow:

```
Victim
        │
        ▼
Initial POST
        │
        ▼
Server Response
        │
        ▼
Victim Identifier / Session ID
        │
        ▼
Repeated POST Requests
```

The response:

```
HckDcK0czXjaq48jVHNn|qIOuKk7U|http://79.124.78.197/index.php|
```

appears to contain a session or victim identifier used during the command-and-control process.

---

# Attack Timeline

| Time (UTC) | Event |
|------------|-------|
| 17:35 | First observed Koi Stealer command-and-control activity |
| 17:35 | POST request to `/foots.php` |
| 17:35 | GET request to `/index.php?id=&subid=qIOuKk7U` |
| 17:35 | Server returns session identifier |
| 17:35 | Continued POST requests to `/index.php` |
| 17:35 | Repeated beaconing between victim and C2 server |

---

# Evidence Correlation

## IDS Alerts

- Koi Stealer C2 Check-in
- Multiple detections associated with the victim host
- 48 alerts indicating repeated communication

---

## Network Traffic

- Repeated HTTP POST requests
- Persistent communication with the same external IP
- Session identifier exchange
- Multiple command-and-control requests

---

## Windows Artifacts

- SMB
- NTLMSSP
- LDAP
- Windows hostname
- User account identification

---

# Executive Summary

On **2024-09-04 at approximately 17:35 UTC**, a Windows workstation (`DESKTOP-RNVO9AT`) used by **Andrew Fletcher** exhibited command-and-control activity consistent with **Koi Stealer** malware. Packet analysis validated IDS alerts showing repeated HTTP communication between the compromised workstation and **79.124.78.197**, including multiple POST requests and a server-issued session identifier. The observed network behavior is consistent with malware beaconing and ongoing C2 communication.

---

# Indicators of Compromise (IOCs)

## Victim

| Item | Value |
|------|-------|
| IP Address | 172.17.0.99 |
| MAC Address | 18:3d:a2:b6:8d:c4 |
| Hostname | DESKTOP-RNVO9AT |
| Windows User | afletcher |
| Full Name | Andrew Fletcher |

---

## External IP

```
79.124.78.197
```

---

## URLs

```
POST /foots.php
GET /index.php?id=&subid=qIOuKk7U
POST /index.php
```

---

## Malware

```
Koi Stealer
```

---

# Malware Extraction

No malware binaries were recoverable from the provided packet capture.

Therefore, no SHA256 hash could be calculated.

---

# Detection Techniques Used

- IDS Alert Analysis
- HTTP Stream Analysis
- SMB Analysis
- LDAP Analysis
- IOC Extraction
- Command-and-Control Analysis
- Evidence Correlation

---

# Lessons Learned

- Begin investigations by reviewing IDS alerts before examining raw packet data.
- Validate IDS detections using packet evidence rather than relying solely on alert descriptions.
- Repeated HTTP POST requests to a single external server can indicate command-and-control beaconing.
- Session identifiers returned by the server may represent victim registration or campaign tracking.
- External threat intelligence can enrich an investigation, but conclusions should distinguish between evidence observed in the PCAP and contextual information from outside sources.

---

# Skills Practiced

- Wireshark
- Network Traffic Analysis
- Incident Response
- HTTP Analysis
- IDS Validation
- IOC Extraction
- C2 Investigation
- Evidence Correlation