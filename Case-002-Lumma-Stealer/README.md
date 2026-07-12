# Case 002 - Lumma Stealer Investigation (Malware Traffic Analysis)

> **Platform:** Malware-Traffic-Analysis.net  
> **Date:** 2026-01-31 Exercise  
> **Difficulty:** Easy  
> **Malware Family:** Lumma Stealer

---

# Objective

Investigate a packet capture (PCAP) generated from a Security Operations Center (SOC) alert related to Lumma Stealer and identify the compromised Windows host along with the associated user.

---

# Scenario

The SOC generated an alert for:

- **Alert:** ET MALWARE Lumma Stealer Victim Fingerprinting Activity
- **External IP:** `153.92.1.49`
- **Protocol:** TCP
- **Alert Time:** 2026-01-27 23:05 UTC

The objective was to determine:

- Infected Windows client
- MAC Address
- Hostname
- User Account
- Full Name
- Malicious Domain

---

# Environment

| Item | Value |
|------|-------|
| LAN | 10.1.21.0/24 |
| Domain | win11office.com |
| Active Directory | WIN11OFFICE |
| Domain Controller | WIN-LU4L24X3UB7 |
| Gateway | 10.1.21.1 |

---

# Initial Indicators of Compromise (IOCs)

| Indicator | Value |
|-----------|-------|
| External IP | 153.92.1.49 |
| Protocol | TCP |
| Destination Port | 443 |
| Malware | Lumma Stealer |

---

# Investigation

## Step 1 - Identify the Infected Host

### Display Filter

```wireshark
ip.addr == 153.92.1.49
```

### Observation

Traffic between the internal host `10.1.21.58` and the external IP `153.92.1.49` was observed.

This confirmed the infected Windows client.


## Step 2 - Identify the MAC Address

The Ethernet header of packets originating from the infected host revealed:

```
00:21:5d:c8:0e:f2
```


## Step 3 - Identify the Hostname

During analysis of Windows network traffic (DHCP/NBNS), the hostname was identified as:

```
DESKTOP-ES9F3ML
```


## Step 4 - Identify the Windows User

Kerberos authentication traffic revealed the authenticated Windows account.

### Display Filter

```wireshark
kerberos.CNameString
```

Result:

```
gwyatt
```


## Step 5 - Identify the User's Full Name

Using **Find Packet** on the surname **Wyatt** within packet details revealed the user's full name.

Result:

```
Gabriel Wyatt
```


## Step 6 - Identify the Malicious Domain

The malicious IP address alone does not indicate the domain name.

During inspection of the TLS Client Hello, the **Server Name Indication (SNI)** extension revealed:

```
whitepepper.su
```

The same domain was also observed while reviewing the HTTP stream, confirming the association.

# Final Findings

| Item | Value |
|------|-------|
| Malware | Lumma Stealer |
| External IP | 153.92.1.49 |
| Malicious Domain | whitepepper.su |
| Victim IP | 10.1.21.58 |
| MAC Address | 00:21:5d:c8:0e:f2 |
| Hostname | DESKTOP-ES9F3ML |
| User Account | gwyatt |
| Full Name | Gabriel Wyatt |

---

# Detection Workflow

```
SOC Alert
      │
      ▼
IOC (153.92.1.49)
      │
      ▼
Victim IP
      │
      ▼
MAC Address
      │
      ▼
Hostname
      │
      ▼
Kerberos Authentication
      │
      ▼
Windows User
      │
      ▼
Full Name
      │
      ▼
TLS / HTTP
      │
      ▼
Malicious Domain
```

---

# Skills Practiced

- Wireshark Packet Analysis
- IOC Pivoting
- TCP Stream Analysis
- HTTP Analysis
- TLS Inspection
- Kerberos Analysis
- Host Identification
- Windows Authentication Analysis
- Digital Forensics
- Incident Investigation

---

# Lessons Learned

- Began the investigation by pivoting from the known IOC rather than manually browsing packets.
- Used multiple protocols to validate findings instead of relying on a single source.
- Learned that Kerberos traffic is useful for identifying authenticated Windows users.
- Discovered that TLS Server Name Indication (SNI) can reveal the destination domain even when DNS evidence is unavailable.
- Confirmed the malicious domain using both TLS and HTTP traffic, increasing confidence in the findings.
- Reinforced the importance of correlating evidence across different network protocols during an investigation.

---

# Tools Used

- Wireshark
- Malware-Traffic-Analysis.net