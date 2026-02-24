# Detection Rules Repository

This folder contains production-ready detection rules for common attack techniques. All rules follow the **Sigma** format (vendor-agnostic) and include Splunk SPL equivalents where applicable.

---

## 📦 What's Included

```
detection-rules/
├── README.md
├── sigma/
│   ├── ssh-brute-force.yml
│   ├── powershell-obfuscation.yml
│   └── lateral-movement.yml
└── splunk/
    └── queries.spl
```

---

## 🔍 Rule Catalog

### 1. SSH Brute Force (`sigma/ssh-brute-force.yml`)
- **Technique:** T1110 (Brute Force)
- **Logic:** >5 failed SSH attempts from single source IP within 10 minutes
- **Use case:** Detect credential stuffing, automated password guessing
- **Platform:** Sigma (ELK, Splunk, QRadar compatible)
- **Severity:** Medium

### 2. Suspicious PowerShell (`sigma/powershell-obfuscation.yml`)
- **Technique:** T1059.001 (PowerShell)
- **Logic:** PowerShell execution with `-EncodedCommand`, `-e`, or `-Enc` flags, excluding benign help commands
- **Use case:** Catch malware using PowerShell for download/execution
- **Platform:** Sigma (requires Windows Event 4688 or Sysmon)
- **Severity:** High

### 3. Pass-the-Hash Lateral Movement (`sigma/lateral-movement.yml`)
- **Technique:** T1550.002 (Pass the Hash)
- **Logic:** NTLM network logons (Event 4624, Logon Type 3) from source workstation not equal to target
- **Use case:** Detect credential reuse across systems
- **Platform:** Sigma (Windows Security Event Log)
- **Severity:** High

---

## 🛠️ Usage Instructions

### For ELK (Elastic Stack)

1. Navigate to **Kibana → Security → Alerts → Create Rule**
2. Choose **Custom query rule**
3. Paste the KQL equivalent (see individual rule files for queries)
4. Set severity and scheduling (e.g., "Run every 5 minutes")
5. Add action: email, Slack webhook, or index into `.siem-alerts`

### For Splunk

1. Open **Splunk Web → Search & Reporting**
2. Create a new alert:
   - Search string: copy from `splunk/queries.spl`
   - Schedule: "Run every 5 minutes"
   - Trigger condition: "Number of results > 0"
   - Actions: send email, create notable event
3. Save as a **real-time alert** or **scheduled search**

### For Sigma Translation

Use `sigmac` (Python tool) to convert to SIEM-specific query:

```bash
# Install sigmac
pip install sigmac

# Convert to Splunk SPL
sigmac -t splunk -c sigma/ssh-brute-force.yml

# Convert to Elasticsearch KQL
sigmac -t es-qs -c sigma/powershell-obfuscation.yml
```

---

## ✅ Testing Your Rules

Before deploying to production:

1. **Load test data** into your SIEM (use sample logs or generate attacks)
2. **Run the query manually** and verify it triggers on attack, not on benign traffic
3. **Check performance** – avoid queries that scan entire index every minute
4. **Document false positives** and adjust thresholds
5. **Add exclusions** for known-good IPs/users

Example test for SSH rule:

```kql
/* Simulate brute force from 192.168.1.200 */
@timestamp >= now-15m AND
source.ip: "192.168.1.200" AND
message: "Failed password for"
```

---

## 📊 Rule Development Checklist

- [ ] Maps to MITRE ATT&CK technique
- [ ] Reasonable false positive rate (<5% in test environment)
- [ ] Clear documentation (author, date, severity, references)
- [ ] Tested against known benign traffic
- [ ] Alerting action configured (email/SIEM ticket)
- [ ] Runbook created (what analyst should do when alert fires)
- [ ] Review scheduled (30 days) for tuning

---

## 📚 References

- **Sigma HQ:** https://github.com/SigmaHQ/sigma
- **MITRE ATT&CK:** https://attack.mitre.org/
- **Splunk SPL Best Practices:** https://docs.splunk.com/Documentation/Splunk/latest/AdvancedDev/WorkInProgress
- **Elastic Detection Rules:** https://www.elastic.co/guide/en/security/current/rules-ui-create.html

---

*Contributions welcome – fork and submit PRs for new rules.*

*Last updated: 2026-02-23*
