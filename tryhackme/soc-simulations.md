# TryHackMe: SOC Simulations – Write-Up

**Rooms:** SOC L1 Alert Triage | SOC L1 Alert Reporting | Junior Security Analyst Intro
**Track:** SOC Level 1
**Difficulty:** Easy
**Completion Dates:** 2026-02-22 – 2026-02-23
**Badges Earned:**
- `soc-sim-first-alert-closed` (rare, 6.9%)
- `soc-sim-first-scenario-completed` (rare, 5.7%)
- `soc-sim-100-percent-true-positive-rate` (rare, 4.7%)

---

## 🎯 Room Objectives

- Experience **day-in-the-life** of a SOC analyst
- Practice **alert triage**: filtering false positives, escalating real incidents
- Learn **incident reporting**: clear documentation, timeline, impact assessment
- Understand **SOC processes** and stakeholder communication

---

## 📋 What the Simulations Involved

### Scenario Setup
- Simulated environment with pre-generated security alerts
- SIEM interface (custom THM platform) with search and investigation tools
- Multiple attack scenarios: malware infection, brute force, data exfiltration attempts
- Time pressure: need to close X alerts within Y minutes (like real SOC)

### Alert Types Encountered

| Alert | Source | Severity | True/False? | My Decision |
|-------|--------|----------|-------------|-------------|
| Multiple SSH failures from 10.10.10.200 | auth.log | Medium | True | Escalate – brute force |
| Legit admin login from known IP | Windows Security | Low | False | Close - expected |
| Internal host connecting to known C2 IP | firewall | High | True | Investigate – malware |
| DNS query to suspicious TLD | dns.log | Medium | True | Triage – C2 beacon |
| Port scan from scanning tool IP | ids | Medium | False | Close - testing |
| Account lockout for user jsmith | Windows Security | Medium | True | Escalate – possible compromise |

---

## 🔍 My Triage Methodology

I developed a **systematic approach** that led to 100% accuracy:

### Step 1: Categorize by Source
| Source | Typical Meaning | Investigation Depth |
|---------|----------------|---------------------|
| `auth.log` / Windows 4625 | Authentication issues | Check `count` from same IP/user |
| `firewall` denials | Blocked traffic | Usually benign unless targeted |
| `dns.log` | DNS queries | Look for DGA patterns, rare TLDs |
| `ids` / `suricata` | Known exploit/signature | High confidence if signature is reliable |
| `edr` process alerts | Suspicious process tree | Correlate with network activity |

### Step 2: Reduce Noise Fast

**Common false positives I identified quickly:**
- Single failed login vs. repeated failures (threshold: >5)
- Vulnerability scanners: known IP (`10.10.10.5`), User-Agent `nmap`
- Time-based: after-hours backup jobs (scheduled task: `backup.exe`)
- Legitimate admin tools: `PsExec.exe` from admin workstation (but watch for use by non-admin users)

**Quick filters in SIEM:**
```kql
# Exclude known benign sources
NOT source.ip: ("10.10.10.5" OR "10.10.10.10")
# Exclude known admin users
NOT user.name: ("admin1" OR "sysadmin")
```

### Step 3: Correlate Across Logs

If alert says "Suspicious process launched":
1. Find `process.id` in syslog → what command line?
2. Check network connections from same `host` → external IP?
3. Check authentication events from same `host` → prior compromise?
4. Check DNS queries → any known malicious domains resolved?

**Example successful correlation:**
- Alert: "PowerShell download cradle detected" (EDR)
- Search: `process.name=powershell.exe host=10.10.10.30`
- Found: `CommandLine: -EncodedCommand <base64>`
- Correlated: same host made DNS query to `malicious.xyz` 10 seconds earlier
- Decision: **True Positive – malware execution**

---

## 📊 How I Achieved 100% True Positive Rate

### Triage Principles Applied

1. **Never trust a single alert.** Always investigate context.
2. **Look for multi-step attacks.** Malware doesn't appear in isolation – expect prior recon, C2, or lateral movement.
3. **Use the Pyramid of Pain.** Focus on high-value IOCs: domain names, file hashes, TTPs (not just IPs).
4. **Prioritize by impact.** User workstation > server > non-critical asset.
5. **Escalate uncertain cases** – better safe than sorry in a simulation.

### Common Pitfalls I Avoided

| Pitfall | How I Avoided |
|---------|---------------|
| Clicking "Close" on every alert | Investigated each for at least 30 seconds |
| Focusing only on severity | Looked at context; sometimes "Low" was true positive |
| Assuming tool accuracy | Verified with raw logs, didn't trust alert reason blindly |
| Time pressure rushing | Prioritized: high-confidence true positives first, then borderline |

---

## 📝 Incident Reporting (What I Did Right)

When escalating, I wrote structured reports:

```
Title: Brute Force Campaign Against SSH Servers

Summary: Source IP 10.20.30.40 attempted >50 SSH logins against
prod-web-01 and prod-db-01 between 14:00-14:30 UTC.

Evidence:
- auth.log shows: "Failed password for root" (48 times), "Invalid user admin" (12 times)
- Source IP originates from AS12345 (known VPS provider)
- No successful logins observed

Impact: unsuccessful, but indicates targeted attack

Recommendation: Block source IP at firewall, monitor for credential stuffing
```

**Why it works:** clear, factual, actionable. SOC managers can triage quickly.

---

## 💡 Key Takeaways

1. **Alert fatigue is real.** Simulated alerts included many false positives to train filtering.
2. **The real skill is investigation, not clicking buttons.** SIEM is just a tool – your brain does the triage.
3. **Documentation matters.** Good reports build trust with incident response team.
4. **Time management is critical.** I closed ~15 alerts in 20 minutes by using saved searches and filters.
5. **Know your environment.** In the sim, I learned which IPs were "internal admin" vs "scanner" vs "attacker."

---

## 🎯 Next Learning Steps

- **Real-world log ingestion:** Get my local SIEM lab ingesting Windows Event logs to practice similar triage
- **Build a triage checklist** – a one-page cheat sheet for common false positives
- **Practice under time pressure** – simulate 30 alerts in 15 minutes using my ELK lab
- **Learn Elastic/ Splunk Saved Searches** that auto-triage (e.g., "Known false positive IPs" dashboard)

---

## 📸 Screenshots

*(Placeholder – to add)*

- [ ] SOC Simulations room completion badge
- [ ] 100% True Positive Rate badge
- [ ] Sample SIEM investigation timeline
- [ ] Example incident report (from room)

---

*These simulations validated my alert triage methodology. Now I can approach real SOC alerts with confidence.*

*Related artifacts: detection rules in `../detection-rules/`, SIEM lab in `../siem-lab/`.*
