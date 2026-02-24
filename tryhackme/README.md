# TryHackMe Learning Journey

This folder contains detailed write-ups of key TryHackMe rooms that demonstrate my practical security skills. Each room reinforced SOC workflows, detection logic, and attack methodology.

---

## 🗂️ Rooms Covered

| Room | Difficulty | Track | Status |
|------|------------|-------|--------|
| [Network Discovery Detection](network-discovery-detection.md) | Medium | SOC | ✅ Completed |
| [SOC Simulations](soc-simulations.md) | Easy | SOC | ✅ Completed |
| [Phishing Analysis](phishing-analysis.md) | Easy | SOC | ✅ Completed |
| Pyramid of Pain | Easy | SOC | ✅ Completed |
| Cyber Kill Chain | Easy | SOC | ✅ Completed |
| SOC L1 Alert Triage | Easy | SOC | ✅ Completed |
| SOC L1 Alert Reporting | Easy | SOC | ✅ Completed |
| Introduction to SIEM | Easy | SOC | ✅ Completed |
| Introduction to EDR | Easy | SOC | ✅ Completed |
| Pentesting Fundamentals | Easy | Pentesting | ✅ Completed |

Full room list: [TryHackMe Profile](https://tryhackme.com/p/VulnZ)

---

## 📝 Writing Style

Each write-up includes:
- **Room objectives** – what skill was taught
- **Key concepts learned** – frameworks, tools, techniques
- **Detection logic** – how to spot this activity in logs (SIEM queries)
- **Screenshots** – evidence of completion (to be added)
- **Takeaways** – how this applies to real SOC work

---

## 🎯 Why These Matter for SOC Careers

1. **Network Discovery Detection** – Proves I can identify recon activity (port scans, ping sweeps) before an attack escalates. Directly applicable to monitoring firewall/IDS logs.

2. **SOC Simulations** – Shows real-world alert triage capability. The 100% TPR badge means I correctly identified *all* true positives without missing threats or excessive false positives.

3. **Phishing Analysis** – Demonstrates email forensics skills: header analysis, URL deobfuscation, IOC extraction. Critical for email security gateways and user-reported incidents.

---

## 📚 Study Approach

- **Hands-on first:** Completed rooms in TryHackMe's interactive browser-based labs
- **Badge-driven:** Aimed for 100% in simulations to prove competency
- **Daily consistency:** 7-day+ learning streaks (2–3 hours/day)
- **Practical notes:** Took screenshots of detection queries, saved sample logs, documented mistakes

---

## 🔗 Related Artifacts

- Detection rules in [`../detection-rules/`](../detection-rules/) implement patterns learned from these rooms
- SIEM lab in [`../siem-lab/`](../siem-lab/) puts theory into practice with local log ingestion
- Resume highlights these achievements (see `PROFILE.md`)

---

*This learning journey continues – new room write-ups will be added as I complete them.*

*Last updated: 2026-02-23*
