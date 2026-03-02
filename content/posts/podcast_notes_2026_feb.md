---
title: "Podcast Notes: February 2026"
date: 2026-03-01T01:00:00
tags:
  - podcasts

categories:
  - Networking
  - Security

draft: false
url: "podcasts-02-2026"
summary: "Notes from Podcasts listened to February 2026"
description: "Notes from Podcasts listened to February 2026"

---

# Below are some interesting points I found from podcasts listened to during February 2026


---

# 🎧 Security Now!

---

## SN 1061

### Hardware & Infrastructure

- **RAM pricing impacts enterprise firewalls**
  - Rising memory prices will directly affect enterprise firewall vendors like Fortinet and Palo Alto.
  - Modern NGFW platforms rely heavily on RAM for session tables, inspection, logging, and AI features.
  - Hardware BOM cost increases may result in higher appliance pricing or margin pressure.

---

### AI & Software Supply Chain

- **Anthropic’s Python strategy**
  - Strong Python support is a smart long-term investment.
  - AI reviewing PyPI updates could help identify malicious package injections earlier.
  - Signals increasing AI involvement in supply chain security.

- **Grubhub breach**
  - Customer data exposed.
  - Reused authentication tokens from previous breaches.
  - Reinforces importance of:
    - Token invalidation
    - Proper session lifecycle management
    - Credential hygiene

---

### PKI / Certificates

- **Let’s Encrypt testing 6-day certificates**
  - Extremely short-lived domain/IP certificates.
  - Goal: reduce value of stolen certs.
  - Encourages automation of certificate lifecycle.

---

### Malware – GhostPoster Extension

- Malware discovered across multiple browser ecosystems:
  - Microsoft Edge
  - Firefox
  - Google Chrome
- Behavior:
  - Loader hidden in **plaintext code inside PNG file**
  - Delays activation for days
  - Eventually contacts C2 server to download JavaScript payload
  - Hijacks affiliate links for monetization
- Some malicious extensions existed for **5+ years** before detection.
- Demonstrates:
  - Supply chain risk in browser extension ecosystems
  - Delayed-execution malware techniques
  - Long dwell times in consumer environments

---

## SN 1062

### Government & Policy

- **CISA funding uncertainty**
  - Dependent on congressional approval.
  - Highlights fragility of federal cybersecurity initiatives.

- **Ireland legalizes expanded government data access**
  - Raises slippery-slope concerns:
    - Could governments eventually require monitoring apps on devices?
  - Expands lawful interception authority.

- **Microsoft recovery keys**
  - Microsoft turns over stored recovery keys with valid warrants.
  - Users can configure systems so recovery keys are NOT stored in Microsoft accounts.

---

### Networking

- **QoS Reality Check**
  - There is effectively **no end-to-end QoS on the public internet**.
  - Best practice:
    - Preserve DSCP markings across domains.
    - Apply QoS policy at congestion points.
  - No need to apply policy everywhere — apply it where it matters.

---

### Professional Development

- **HN813 – What Network Engineers Should Know About Software Developers**
  - Communication is critical.
  - Assume competence in others.
  - Avoid blame culture.
  - Focus on shared problem-solving.

---

### Vulnerabilities & Industry Changes

- **Notepad++ major flaw**
  - Vulnerability that could allow remote compromise.

- **Microsoft ending NTLM support**
  - Push toward modern authentication (Kerberos / OAuth / Entra ID).
  - Significant enterprise impact.

- **Telnet exposure**
  - Reminder: ensure legacy services are not publicly accessible.

- **AI agents**
  - Scammers and attackers are actively preparing to exploit AI-driven automation tools.

---

## SN 1063

### Notepad++ Supply Chain Attack

- Update server compromised.
- Targeted attack attributed to Chinese actors.
- Allowed full system compromise via malicious update delivery.
- Highlights:
  - Software update channels are high-value targets.
  - Even trusted tools can become attack vectors.

---

### Curl Ends Bug Bounty Program

- AI-generated vulnerability submissions overwhelmed maintainers.
- Highlights systemic issues:
  - Open source relies heavily on volunteer labor.
  - AI is increasing noise-to-signal ratio.
  - Bug bounty systems need reform in AI era.

---

### AI Tooling Risk

- **Gemini in VS Code incident**
  - Bug caused deletion of entire project directories.
  - Reinforces:
    - AI tools must be sandboxed.
    - Guardrails are critical when granting file-system permissions.

---

## SN 1064

### GDPR Enforcement Reality

- ~$4B in fines issued under GDPR.
- Only ~$20M collected.
- Most large fines are tied up in appeals.
- Enforcement bottlenecks weaken regulatory deterrence.
- Raises question:
  - Does regulation without timely enforcement reduce impact?

---

# 🎙 Packet Pushers / Heavy Networking

## Understanding ASICs for Network Engineers (Peet Lumbis)

### ASIC Fundamentals

- ASIC = Application-Specific Integrated Circuit.
- Purpose-built hardware.
- Cannot be reprogrammed like CPUs.
- Designed with a fixed **packet processing pipeline**.
  - Defines exactly:
    - What operations occur
    - In what order
    - On which fields

### Vendor Tradeoffs

All switching ASICs perform similar core functions:
- Forwarding
- ACL processing
- Buffering
- QoS

Differences come from optimization tradeoffs:
- Latency
- Power efficiency
- Feature depth
- Table sizes
- Programmability (limited)

Key takeaway:
Hardware design is always compromise-driven.

---

# 🛡 Fortinet Webinar
## The Art of War: Automating Security Operations  
SE: Jason Palm

> “If you know yourself and the enemy, you will not fear the result of a hundred battles.” — Sun Tzu

---

### Core Principle

**Defense in depth remains foundational.**

Protect:
- Perimeter
- Endpoint (EDR)
- Email
- Internal monitoring

---

### Demo Summary

Attack lifecycle demonstration:

1. User installs malware.
2. Kali Linux connects to infected host.
3. Metasploit used.
4. Meterpreter session established.
5. Attacker:
   - Migrates to system process (PID move)
   - Executes commands
   - Gathers system info
   - Launches shell / PowerShell

---

### Cyber Kill Chain

Recon → Weaponize → Deliver → Exploit → Install → C2 → Actions

Recommendation:
- Overlay your current services and controls onto the kill chain.
- Identify coverage gaps.
- Focus security investment where coverage is weakest.

---

### Fortinet Platform Notes

- New FAZ (FortiAnalyzer) versions support more third-party integrations.
- FortiSIEM:
  - Aggregates logs across enterprise.
  - Enables enterprise-grade SOC capabilities.
  - Managed services can bridge staffing gaps short term.

---

# 🔎 Cross-Theme Observations (February 2026)

- AI is reshaping:
  - Software development
  - Bug bounties
  - Attack surfaces
- Supply chain attacks are increasing in sophistication.
- Short-lived credentials and automation are becoming essential.
- Regulation without enforcement is ineffective.
- Defense-in-depth remains the only sustainable strategy.
- Hardware design (ASICs) reflects engineering tradeoffs — just like security architectures.

---