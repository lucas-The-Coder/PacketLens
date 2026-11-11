# PacketLens - Traffic Analyzer

[Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
[Scapy](https://img.shields.io/badge/Scapy-2.5.0+-green.svg)
[MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1071%20%7C%20T1071.001%20%7C%20T1071.004-red.svg)
[License](https://img.shields.io/badge/License-MIT-lightgrey.svg)
[Status](https://img.shields.io/badge/Status-Complete-brightgreen.svg)

A Python PCAP analyzer that turns raw network traffic into clear SOC findings. It extracts DNS queries, HTTP hosts, and detects possible beaconing behavior from periodic connections.

Built with Scapy. Works on any `.pcap` file. No live capture needed.

> **One-line summary for GitHub:** `PacketLens | Python PCAP analyzer for SOC traffic triage. Extracts DNS queries and HTTP hosts, detects beaconing with regular intervals to flag possible C2.`

---

## Why PacketLens

SOC analysts spend a lot of time opening PCAPs in Wireshark and manually looking for suspicious domains, hosts, and patterns [1]. PacketLens automates the first pass.

Instead of scrolling through hundreds of packets, you run one command and get:
- What domains were queried
- What HTTP hosts were contacted
- What IPs look like they are being beaconed to

That first pass is what makes triage faster. You still investigate, but you start with a short list, not a raw capture. This is the same workflow used in SOC Tier 1 triage: reduce noise, surface candidates, then investigate [2].

PacketLens is intentionally offline and file-based. It does not sniff live traffic. You capture with `tcpdump`, Wireshark, or Zeek, then analyze the file here. This keeps it safe to run on a workstation without promiscuous mode or root privileges for live capture [1].

---

## What It Detects

| Check | What It Finds | Severity | Why It Matters |
| :--- | :--- | :--- | :--- |
| **DNS Query** | Every unique DNS query in the PCAP | INFO | Shows what domains a host tried to resolve. Useful for spotting DGA, typo-squatting, or known bad domains. DNS is often used for C2 because it is rarely blocked [3]. |
| **HTTP Host** | Every unique `Host:` header in HTTP traffic | INFO | Shows what web servers were contacted, even if DNS is not in the capture. Useful for finding cleartext HTTP and unexpected hosts. HTTP is a common C2 channel because it blends with normal web traffic [3]. |
| **Beaconing** | Same destination IP, 5+ packets, with similar intervals (within 5 seconds) | HIGH | A common C2 pattern. Malware calling home every 10 seconds looks normal alone, but regular intervals over time are a strong beaconing signal [4][5]. |

---

## MITRE ATT&CK Mapping

PacketLens maps to Command and Control tactics in MITRE ATT&CK [3]:

| Detection | MITRE Technique | Description |
| :--- | :--- | :--- |
| DNS Query extraction | **T1071.004 - DNS** | Adversaries may communicate using DNS to avoid detection, by using the DNS protocol to encode C2 data [3]. |
| HTTP Host extraction | **T1071.001 - Web Protocols** | Adversaries may communicate using HTTP/HTTPS to blend with normal web traffic and bypass firewalls [3]. |
| Beaconing detection | **T1071 - Application Layer Protocol** + **T1095 - Non-Application Layer Protocol** | Periodic callbacks to C2, often at fixed intervals, to check for new instructions [3][4]. |

This mapping is why the tool is useful in a SOC. It does not just list packets, it lists packets that map to known adversary behavior.

References for this mapping are in the [References](#references) section at the bottom.

---

## How It Works
