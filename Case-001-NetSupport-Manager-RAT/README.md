# Case 001 – NetSupport Manager RAT Investigation

## Objective

Investigate suspicious traffic associated with NetSupport Manager RAT and identify the compromised Windows host using Wireshark.

---

## Scenario

A SOC generated multiple alerts indicating communication with:

45.131.214.85:443

The objective was to identify:

- Infected IP
- MAC Address
- Hostname
- Windows User
- Full Name of User

---

## Initial Indicators of Compromise (IOCs)

| Indicator | Value |
|-----------|-------|
| External IP | 45.131.214.85 |
| Protocol | TCP |
| Port | 443 |
| Malware | NetSupport Manager RAT |

## Step 1 – Identify the Compromised Host

### Filter Used

ip.addr == 45.131.214.85

### Observation

Traffic between 10.2.28.88 and 45.131.214.85 was identified.

Following the TCP stream revealed HTTP POST requests with the User-Agent:

NetSupport Manager/1.3

This confirmed communication with the command-and-control server.

### Result

Victim IP:
10.2.28.88

## Step 2 – Identify the Hostname

### Initial Attempt

I first inspected DHCP traffic.

The DHCP Request packet contained:

DESKTOP-TEYQ2NR

Later I learned that filtering NBNS provides a faster method for identifying Windows hostnames.

### Result

Hostname:
DESKTOP-TEYQ2NR

## Step 3 – Identify the User

### Filter Used

kerberos.CNameString

### Observation

Kerberos authentication traffic revealed the Windows user account.

User: brolf

## Step 4 – Identify Full Name

Searching packet details for: Rolf

revealed the user's full name.

Result: Becka Rolf

# Final Findings

| Item | Value |
|------|-------|
| Malware | NetSupport Manager RAT |
| External IP | 45.131.214.85 |
| Victim IP | 10.2.28.88 |
| MAC Address | 00:19:d1:b2:4d:ad |
| Hostname | DESKTOP-TEYQ2NR |
| User | brolf |
| Full Name | Becka Rolf |

# Lessons Learned

This investigation helped me understand how to pivot from an IOC rather than manually browsing packets.

Key takeaways:

- Use IOC filtering first.
- Follow TCP streams to verify suspicious traffic.
- NBNS is an efficient way to identify Windows hostnames.
- Kerberos traffic reveals authenticated usernames.
- Packet analysis is an investigation process rather than memorizing filters.

# Skills Practiced

- Wireshark
- IOC Pivoting
- TCP Stream Analysis
- HTTP Analysis
- NBNS
- DHCP
- Kerberos
- Incident Investigation