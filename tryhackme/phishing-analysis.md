# TryHackMe: Phishing Analysis Fundamentals & Emails in Action – Write-Up

**Rooms:** Phishing Analysis Fundamentals | Phishing Emails in Action
**Track:** SOC Level 1
**Difficulty:** Easy
**Completion Date:** 2026-02-22
**Tools Used:** Email header analyzer, URL sandboxing, VirusTotal, manual inspection

---

## 🎯 Room Objectives

- Understand email structure: headers, body, attachments, links
- Learn to extract **indicators of compromise (IOCs)**
- Identify phishing red flags: spoofed addresses, mismatched URLs, urgent language
- Practice analysis workflow: header → links → attachments → verdict

---

## 📧 Email Anatomy Refresher

```
Received: from mail.example.com (10.0.0.1) by mx.google.com...
📌 Shows the email's path – look for mismatched hops

Authentication-Results: spf=pass (google.com)...
📌 SPF/DKIM/DMARC results – did the sender authenticate?

From: "Support" <support@amaz0n-security.com>
📌 Display name ≠ actual domain (amaz0n vs amazon) – red flag

Subject: Urgent: Your account locked!
📌 Urgency + threat = common phishing tactic

Body: Click here to verify: http://bit.ly/2xYz (obfuscated link)
📌 Hover to reveal actual destination

Attachment: invoice.pdf.exe (double extension)
📌 Executable disguised as PDF – malicious
```

---

## 🔍 My Analysis Workflow

### Step 1: Header Analysis

**Key headers to check:**

| Header | What to Look For |
|--------|------------------|
| `Received` | Does the origin match claimed sender? Look for foreign IPs |
| `From` / `Reply-To` | Are they the same? If not, suspect |
| `Return-Path` | Where bounces go – often different in spoofed emails |
| `Authentication-Results` | SPF/DKIM pass/fail – fail = suspicious |
| `X-Mailer` / `User-Agent` | Legitimate services use known agents (Outlook, Gmail) |

**Example finding:**
```
From: Netflix <no-reply@netflix-billing.info>
Received: from 185.130.5.12 (Russia)
SPF: fail (domain does not exist)
→ High confidence phishing
```

---

### Step 2: Link Inspection

**Never click live links!** Use sandbox or view source.

**Techniques:**
- Hover to see `href` (shows actual URL)
- Expand shortened URLs (bit.ly, t.co) via `curl -I <url>` or online expander
- Check domain age (newly registered = suspicious) via WHOIS
- Check URL for **homoglyphs**: amaz0n.com, paypàl.com

**Red flags:**
- IP address in URL (http://192.168.1.1/login.php)
- HTTPS but certificate invalid (still risky)
- Login form on non-brand domain
- URL contains `@` (basic auth in URL) or `//` (path traversal)

---

### Step 3: Attachment Analysis

**File type mismatches:**
- `.pdf.exe` – Windows hides known extensions
- `.docm` – macro-enabled document (common malware dropper)
- `.zip` with `.js` or `.vbs` inside

**Sandbox approach:**
- Upload to VirusTotal (public/private)
- Check Any.run for dynamic behavior
- If no AV hits, still suspicious based on sender/context

---

### Step 4: Content Red Flags

| Pattern | Significance |
|---------|--------------|
| Urgent language ("account suspended", "immediate action") | Creates panic, bypasses rational thinking |
| Too-good-to-be-true offers | Social hook |
| Grammar/spelling errors | Often non-native speakers (typical spam) |
| Unexpected attachment | You didn't order an invoice |
| Mismatched branding | Logo blurry, colors off |

---

## 🧪 Room Walkthrough (Example)

**Email Sample:**
```
From: IT Support <support@microsoft-security-team.com>
Subject: Password Expiry Notification
Body: Your password expires in 24h. Click to reset: http://microsft-login.com/secure
Attachment: None
```

**Analysis:**
1. `From` domain: microsoft-security-team.com – not microsoft.com → **spoofed**
2. Link domain: microsft-login.com – misspelling of Microsoft → **phishing**
3. Urgency: "24h" → classic tactic
4. SPF likely fails (not authorized Microsoft domain)

**Verdict:** Phishing ✅

---

## 📊 IOC Extraction Template

After confirming phishing, Iocreatestructured notes:

```
Phishing Campaign: Microsoft Password Expiry Lure

From Address: support@microsoft-security-team.com
Reply-To: (same or different)
Received IP: 185.130.5.12 (Russia) – submit to AbuseIPDB
Link URL: http://microsft-login.com/secure
Link IP: 104.28.12.34 (hosted by Namecheap)
Whois: Registered 2 days ago, privacy protected
VT Hits: 6/89 (malicious)
Attachment: N/A
Verdict: Phishing – Credential Harvesting
TTP: Social engineering + fake login page
MITRE: T1566.001 (Phishing: Spearphishing Link)
Recommended Action: Block domain/IP at Gateway, user notification, password reset
```

---

## 🛡️ Defensive Recommendations

**For organizations:**
- **Email gateway rules:** Block known phishing domains, scan URLs in real-time
- **DMARC enforcement:** Prevent spoofed emails from reaching inbox
- **User training:** Simulated phishing campaigns (THM has a room for this)
- **Report button:** Make it easy for users to report suspicious emails

**For SOC analysts:**
- **Quick triage:** Use search patterns like `from:*@*.tk OR from:*@*.xyz` for throwaway domains
- **Correlation:** If one user reports phishing, check other users' inboxes for same sender
- **Hunting:** Search DNS logs for lookups to reported phishing domains across the org

---

## 💡 Takeaways

1. **Phishing is still #1 attack vector** – SOC must have clear process for handling user reports
2. **Headers are gold** – they reveal origin even if From is spoofed
3. **Never trust the display name** – always check the actual email address
4. **URL obfuscation is common** – expand, don't just hover
5. **Document IOCs** – blocklists, threat intel sharing, post-incident analysis

---

## 📸 Screenshots

*(Placeholder – add actual room completion screenshots)*

- [ ] Phishing Analysis Fundamentals badge
- [ ] Phishing Emails in Action badge
- [ ] Sample header analysis showing spoofed origin
- [ ] IOC extraction spreadsheet template

---

*Next: Apply this skill in a real environment – practice on user-reported emails if possible. Build a standard report template (see IOC extraction above) for SOC ticket creation.*

*Related artifacts: detection rules for phishing links in `../detection-rules/`.*
