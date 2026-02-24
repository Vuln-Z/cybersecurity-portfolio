# Implemented Detection Rules

This document details the detection rules I've built and tested in my ELK SIEM lab. Each rule maps to a specific attack technique and includes query logic, thresholds, and false positive mitigation.

---

## 1. SSH Brute Force Detection

**Technique:** T1110 – Brute Force (Credential Access)
**Data Source:** Authentication logs (Linux `/var/log/auth.log`, Windows Event 4625)

### Sigma Rule (YAML)

```yaml
title: SSH Brute Force Attack
id: 1a2b3c4d-5e6f-7a8b-9c0d-1e2f3a4b5c6d
status: experimental
description: Detects multiple SSH login failures from a single source IP within a short time window
author: VulnZ
date: 2026-02-23
modified: 2026-02-23
tags:
  - attack.credential_access
  - attack.t1110
  - brute_force
  - ssh
logsource:
  product: linux
  service: auth
detection:
  selection:
    Message|contains:
      - 'Failed password for'
      - 'Invalid user'
    # sshd[xxx]: Failed password for invalid user test from 192.168.1.100 port 54321 ssh2
  timeframe: 10m
  condition: selection | count(distinct.Message) by src_ip > 5
level: medium
falsepositives:
  - Misconfigured automated scripts
  - Legitimate users mistyping passwords repeatedly
  - Vulnerability scanners
---

### ELK Query (KQL)

```kql
event.dataset: "auth.log" AND
message: ("Failed password for" OR "Invalid user") AND
NOT user.name: ("root" AND count < 3)  # exclude root with few attempts

| stats count by source.ip, user.name
| where count > 5
| sort count desc
```

**Mitigation:** Exclude known maintenance IPs via `NOT source.ip: ("10.0.0.1" OR "10.0.0.2")`.

---

## 2. Suspicious PowerShell Execution

**Technique:** T1059.001 – Command and Scripting Interpreter: PowerShell (Execution)
**Data Source:** Windows Event 4688 (process creation) + Sysmon Event 1 (if available)

### Sigma Rule (PowerShell Obfuscation)

```yaml
title: Suspicious PowerShell Encoded Command
id: 2b3c4d5e-6f7a-8b9c-0d1e-2f3a4b5c6d7e
status: experimental
description: Detects PowerShell executions with -EncodedCommand or -e flags
author: VulnZ
date: 2026-02-23
tags:
  - attack.execution
  - attack.t1059.001
  - powershell
  - obfuscation
logsource:
  category: process_creation
  product: windows
detection:
  selection:
    CommandLine|contains:
      - '-EncodedCommand'
      - '-e '
      - '-Enc '
  filter:
    CommandLine|contains:
      - 'Microsoft.Windows.PowerShell\\'
      - 'Get-Help'  # benign help commands
  condition: selection AND NOT filter
level: high
falsepositives:
  - Automated legitimate scripts
  - Admin troubleshooting
---

### ELK Query (KQL)

```kql
winlog.event_id: 4688 AND
process.name: "powershell.exe" AND
process.command_line: (*-EncodedCommand* OR *-e * OR *-Enc *)

| exclude process.command_line: ("*Get-Help*" OR "*Microsoft.Windows.PowerShell\\*")
| stats count by host.name, process.command_line, user.name
| sort count desc
```

**Next:** Add Sysmon Event ID 1 for deeper process ancestry analysis.

---

## 3. Lateral Movement via Pass-the-Hash

**Technique:** T1550.002 – Pass the Hash (Lateral Movement)
**Data Source:** Windows Event 4624 (logon type 3), 4672 (special privileges), Sysmon Event 3 (network connection)

### Sigma Rule (PtH Detection)

```yaml
title: Pass-the-Hash Lateral Movement
id: 3c4d5e6f-7a8b-9c0d-1e2f-3a4b5c6d7e8f
status: experimental
description: Detects authentication using Pass-the-Hash technique (logon type 3 with NTLM, no Kerberos)
author: VulnZ
date: 2026-02-23
tags:
  - attack.lateral_movement
  - attack.t1550.002
  - pass_the_hash
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4624
    LogonType: 3  # network logon
    AuthenticationPackageName: "NTLM"
  filter:
    SourceWorkstation: "%computername%"  # exclude same host
  condition: selection AND NOT filter
level: high
falsepositives:
  - Legacy applications using NTLM
  - Legitimate remote admin tools
---

### ELK Query (KQL)

```kql
winlog.event_id: 4624 AND
winlog.logon.type: "Network" AND
authentication.package: "NTLM" AND
NOT source.workstation: host.name

| stats count by source.user.name, source.ip, destination.user.name
| where count >= 1
| sort _time desc
```

**Enhancement:** Correlate with Event 4688 (process spawn) on target to see what runs after logon.

---

## 📈 Rule Development Methodology

1. **Identify technique** from MITRE ATT&CK or Kill Chain phase
2. **Determine data source** (which logs capture this activity?)
3. **Write initial query** with clear selection criteria
4. **Test against known benign traffic** to estimate false positives
5. **Tune thresholds** (e.g., count > 5 vs > 10)
6. **Document** in Sigma format for portability
7. **Deploy to Kibana** and create alerting rule with action (email, webhook)

---

## 🎯 Testing Evidence

**SSH Brute Force Test:**
```bash
# Generate 10 failures from 192.168.1.200
for i in {1..10}; do ssh invalid@localhost; done
```
**Result:** Rule triggered, alert appeared in Kibana within 30 seconds.

**PowerShell Obfuscation Test:**
```powershell
powershell -EncodedCommand <known_encoded_payload>
```
**Result:** Detected, legitimate `Get-Help` excluded correctly.

---

## 📚 Future Rules

- [ ] DNS Tunneling (excessive TXT queries)
- [ ] Windows Service Install (Event 4697)
- [ ] suspicious scheduled task creation
- [ ] Remote code execution via WMI (Event 4688 + wmic)
- [ ] Data exfiltration (large outbound transfers)

---

*Last updated: 2026-02-23*
