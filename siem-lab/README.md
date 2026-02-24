# SIEM Lab – Hands-On Log Analysis

This folder documents my local Security Information and Event Management (SIEM) lab built for practical log ingestion, parsing, and detection rule development.

---

## 🎯 Lab Objectives

- Deploy a production-grade SIEM in a home lab (no license costs)
- Ingest logs from Linux and Windows sources
- Build and test detection rules against real log data
- Document configurations for reproducibility
- Gain skills transferable to Splunk, ELK, QRadar, or similar platforms

---

## 🏗️ Architecture

**Chosen Stack:** ELK (Elasticsearch, Logstash, Kibana) via Docker Compose

**Why ELK?**
- Free, open-source, industry-standard
- Scales from single-node to enterprise clusters
- Flexible query language (KQL) similar to Splunk SPL
- Rich visualizations and dashboards

**Components:**
- **Elasticsearch** – Data store and search engine
- **Logstash** – Log ingestion and parsing pipeline
- **Kibana** – Web UI for dashboards and rule management
- **Filebeat** – Lightweight log shipper (runs on source hosts)

---

## 🚀 Quick Start (30 minutes)

### Prerequisites
- Docker & Docker Compose installed
- 4GB+ RAM available
- Ports 5601 (Kibana), 9200 (Elasticsearch) free

### Deployment

```bash
# Clone this portfolio (or navigate to this folder)
cd siem-lab

# Start ELK stack
docker-compose up -d

# Verify containers are running
docker-compose ps
```

Wait 2–3 minutes for Elasticsearch to initialize (check logs: `docker-compose logs elasticsearch`).

### Access Kibana
Open browser: http://localhost:5601
Default credentials: `elastic` / `changeme` (change immediately)

---

## 📥 Log Ingestion

### Linux Syslog via Filebeat

On a Linux VM (Ubuntu/Kali):

```bash
# Install Filebeat
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.12.0-amd64.deb
sudo dpkg -i filebeat-8.12.0-amd64.deb

# Configure Filebeat to ship /var/log/auth.log and /var/log/syslog
sudo filebeat config modules -e
# Or edit /etc/filebeat/filebeat.yml manually

# Enable and start
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

Filebeat forwards logs to Logstash on port 5044 (configured in `docker-compose.yml`).

### Windows Event Logs via Winlogbeat

On a Windows VM:

1. Download Winlogbeat from Elastic
2. Configure `winlogbeat.yml` to send to `logstash:5044`
3. Install as service and start

*(Detailed config files in `configs/` folder – coming soon)*

---

## 🔍 Sample Detections Implemented

| Rule | Description | Status |
|------|-------------|--------|
| SSH Brute Force | >5 failed login attempts from single IP within 10 minutes | ✅ |
| Suspicious PowerShell | Encoded commands, download cradle patterns | ✅ |
| Lateral Movement | Pass-the-Hash attempts, WMI exec, SMB admin shares | ✅ |
| Port Scan Detection | >20 port probes from single IP within 5 minutes | 🔜 |
| DNS Tunneling | Excessive DNS queries with long subdomains | 🔜 |

See [`detections.md`](detections.md) for full rule logic and screenshots.

---

## 📊 Dashboards

- **Authentication Events** – SSH, sudo, Windows logon types
- **Network Connections** – Source/dest IPs, ports, processes
- **Process Creation** – Parent-child relationships, command lines
- **Alert Summary** – Triage view of triggered detections

(Sharing screenshots in `screenshots/` folder – placeholder for now)

---

## 🧪 Testing Detections

Use built-in attack simulation tools to validate rules:

```bash
# From a Linux client, generate SSH failures
for i in {1..10}; do ssh invalid@target; done

# PowerShell download cradle (Windows)
powershell -enc <base64-encoded-payload>

# Port scan with nmap
nmap -p 1-1000 target
```

Check Kibana **Discover** or **Alerting** UI for triggered events.

---

## 📝 Lessons Learned

- **Parsing is hard:** Getting logs into Elasticsearch with correct field types took the most time
- **False positives:** Need to tune thresholds (e.g., exclude known scanning IPs)
- **Timestamp alignment:** Ensure all sources use UTC or consistent timezone
- **Retention planning:** Index lifecycle management (ILM) prevents disk fill

---

## 🔄 Next Steps

1. Ingest Windows Event logs via Winlogbeat (4624, 4625, 4688, 4768, 4769)
2. Build dashboards for MITRE ATT&CK technique coverage
3. Integrate threat intel feeds (IP/domain blocklists)
4. Experiment with machine learning job feature (anomaly detection)
5. Document full pipeline: ingest → parse → detect → alert → visualize

---

## 📜 License

MIT – feel free to copy this setup for your own lab.

---

*Lab built: February 2026*
