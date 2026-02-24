# TryHackMe: Network Discovery Detection – Write-Up

**Room:** Network Discovery Detection
**Track:** SOC Level 1
**Difficulty:** Medium
**Completion Date:** 2026-02-22
**Time Spent:** ~1.5 hours
**Badge Earned:** *(room completion)*

---

## 🎯 Room Objectives

- Understand what **network discovery** means in the attacker kill chain
- Identify common discovery techniques: ping sweeps, port scans, OS fingerprinting, DNS enumeration
- Learn how to detect these activities in network logs and SIEM tools
- Correlate multiple low-severity events into a high-confidence detection

---

## 🔍 Key Concepts Learned

### 1. Why Attackers Do Discovery

Before exploiting a target, attackers need to answer:
- **What hosts are alive?** (ICMP ping, ARP scan)
- **What ports are open?** (TCP SYN scan, service version detection)
- **What OS is running?** (TCP/IP stack fingerprinting, TTL/Window size analysis)
- **What user accounts exist?** (LDAP queries, SID enumeration)
- **What shares are available?** (SMB enumeration)

This maps to **Cyber Kill Chain** phase 3: **Reconnaissance** (or **Discovery** in MITRE'sEnterprise ATT&CK: T1016, T1046, T1087, T1135).

### 2. Common Discovery Tools

| Tool | Purpose | Detectable Signature |
|------|---------|----------------------|
| `nmap` | Port scanning, OS detection | SYN packets to many ports, `-sV` probes |
| `ping` / ICMP sweep | Host discovery | ICMP echo requests to multiple IPs |
| `arp-scan` | Local network mapping | ARP requests for many IPs |
| `dnsenum` / `sublist3r` | Subdomain/record enumeration | Many DNS A/AAAA queries, zone transfer attempts |
| `ldapsearch` | AD enumeration | LDAP bind/search for user/computer objects |
| `smbclient` / `net view` | Share enumeration | SMB protocol negotiations, tree connect to `IPC$` |

---

## 📊 Log Sources for Detection

| Log Type | What It Captures | SIEM Integration |
|----------|------------------|------------------|
| **Firewall logs** | Connection attempts (allowed/denied) | Source/dest IP, port, action, timestamp |
| **IDS/IPS** (Suricata, Snort) | Known scan patterns, suspicious payloads | Alert signatures, metadata |
| **Zeek/Bro** | Network protocol analysis | `conn.log`, `http.log`, `dns.log` have full connections |
| **Windows Event** (Firewall) | Outbound connection attempts | Event ID 2004 (allowed), 2005 (blocked) |
| **NetFlow/IPFIX** | Flow summaries (who talked to whom) | Volume-based detection (many flows to different ports) |

---

## 🧪 Hands-On Exercise

In the room, I:
1. Ran an `nmap -p 1-1000 10.10.10.100` scan against a target
2. Captured packets with Wireshark to see the **SYN → SYN/ACK** handshake pattern
3. Viewed the same traffic in **Elastic SIEM** (simulated) with filters:
   ```kql
   network.protocol: "tcp" AND network.transport: "syn"
   | stats count by source.ip, destination.ip, destination.port
   | where count > 20  # port scan threshold
   ```
4. Identified the **scanning IP** by high distinct port count from single source
5. Correlated with **IDS alerts** (Suricata rules: `ET SCAN NMAP TCP scan`)

---

## 🚨 Detection Strategies

### A. Port Scan Detection (Single Source → Many Ports)

**Logic:** Count distinct destination ports from one source IP in short timeframe (e.g., 1 minute). Threshold: >20 unique ports.

**KQL Example:**
```kql
destination.port > 0
| stats dc(destination.port) as unique_ports by source.ip, destination.ip
| where unique_ports > 20
```

**False positives:** Legitimate vulnerability scanners, monitoring tools. Exclude known IPs.

---

### B. Ping Sweep Detection (Single Source → Many Hosts)

**Logic:** Count distinct destination IPs (ICMP echo) from one source in short window.

**Zeek log query:**
```kql
source: "zeek" conn_state: "SF" AND
proto: "icmp" AND
| stats dc(id.resp_h) as hosts_pinged by id.orig_h
| where hosts_pinged > 10
```

---

### C. DNS Enumeration Detection

**Logic:** Many distinct DNS A/AAAA queries for same domain from one source (subdomain brute-forcing).

**DNS log query:**
```kql
query_type: "A" OR query_type: "AAAA"
| stats dc(answers) as distinct_ips by client, query
| where distinct_ips > 5
```

---

## 📈 Correlation Over Single Events

**Important:** No single packet = compromise. But **multiple discovery behaviors** increase risk:

Example correlation:
- Source IP `192.168.1.200`:
  - Scanned 1000+ ports on `10.0.0.5`
  - Then ping-swept the /24 subnet
  - Then made LDAP binds to domain controller

**This pattern = high confidence recon** → trigger high-severity alert.

---

## 🎯 Real-World Application

In my **local SIEM lab**, I've implemented:

- **Elastic detection rule** using the queries above
- **Thresholds tuned** to my lab size (small = lower thresholds)
- **Alert actions**: send to `.siem-alerts` index and email notification
- **Dashboard**: "Discovery Activity" showing top scanning sources per hour

---

## 💡 Takeaways

1. **Detection is about patterns** – not single events
2. **Baseline normal** – know what legitimate scanning looks like in your environment
3. **Tune thresholds** – too low = alert fatigue, too high = miss real attacks
4. **Correlate sources** – combine firewall + IDS + DNS logs for confidence
5. **Enrich with threat intel** – is this source IP known malicious? Check VirusTotal, AbuseIPDB

---

*Completed as part of SOC Level 1 path. Builds directly on Pyramid of Pain and Cyber Kill Chain knowledge.*

*Next: Apply this detection logic to my own lab with real nmap scans.*
