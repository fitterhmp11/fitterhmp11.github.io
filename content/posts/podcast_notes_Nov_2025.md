---
title: "Podcast Notes: November 2025"
date: 2025-11-30T01:00:00
tags:
  - podcasts
categories: 


draft: false
url: "podcasts-11-2025"
summary: "Notes from Podcasts listened to November 2025"
description: "Notes from Podcasts listened to November 2025"
---
# Below are some  interesting points I found from podcasts listened to during November 2025

# November 2025 Podcast Notes
# 📌 Podcast Notes (Organized & Clarified)

---

## 🎧 Security Now! (with Steve Gibson & Leo Laporte)  
**Show:** Security Now!  
**Link:** https://twit.tv/shows/security-now  
**About:** Weekly cybersecurity podcast covering vulnerabilities, attacks, defenses, and security theory.

---

### 🛡️ SN 1049 — DNS Cache Poisoning Returns

**Key Topics**
- **Government vulnerability disclosure laws**
  - **China (2021):** All vulnerabilities must be reported to a government database before vendors.
  - **Impact:** Increase in zero-day usage attributed to PRC-aligned APTs.
  - **Russia (2025):** Considering a similar law (pending approval).

- **Primary intrusion vectors remain unchanged**
  - Remote access compromise
  - Phishing
  - Software vulnerabilities

**DNS Cache Poisoning History**
- **2008 discovery:** Dan Kaminsky identified a critical DNS cache poisoning flaw.
- **How the attack worked:**
  - DNS uses UDP with a transaction ID and source port (32-bit total).
  - Attackers could predict these values and race the legitimate DNS response.
  - If the attacker wins, the resolver cache is poisoned.
- **Original fix (2008):**
  - Better randomization (PRNG) of transaction IDs and source ports.
- **Current concern:**
  - Some modern DNS implementations (BIND, Unbound) have weak PRNG behavior.
  - This reintroduces cache poisoning risk.
- **Note:** DNSSEC is the real long-term fix, but adoption is incomplete.

**Episode Link:** https://www.grc.com/sn/sn-1049.htm

---

### 🛠️ SN 1050 — AI, Supply Chain, and Persistent Backdoors

**Key Topics**
- **Chinese e-buses in Norway**
  - Unauthorized radios discovered.
  - Concerns include surveillance or remote kill-switch capability.

- **OpenAI Aardvark**
  - Autonomous code vulnerability scanner.
  - Designed to find and suggest fixes for software bugs at scale.

- **Cisco router compromises (Australia)**
  - Unpatched routers found with persistent malware.
  - Malicious code survives reboot, indicating deep compromise.

- **GitHub language trends**
  - TypeScript surpassed Python and JavaScript as the most-used language.

**Security Design Insights**
- **Password handling**
  - Client-side hashing (even over TLS) adds protection.
  - Servers must re-hash on receipt.
  - A hashed password is effectively the password if intercepted.
- **Core principle**
  - Never assume transport-layer security alone is sufficient.

**AI & Browser Security**
- **AI-embedded browsers**
  - Rapid innovation introduces privacy and security risks.
- **Prompt injection**
  - Manipulating AI behavior via crafted input.
  - Comparable to SQL injection but targeting LLM logic.
- **Risk example**
  - AI interpreting webpage content as executable user instructions.

**Episode Link:** https://www.grc.com/sn/sn-1050.htm

---

### 📱 SN 1052 — Digital Identity, AI Compute, and Surveillance

**Key Topics**
- **Apple Digital ID**
  - Digital passport support in Apple Wallet.
  - Accepted as REAL ID for domestic flights in some regions.

- **AI cloud compute**
  - Public cloud AI resembles mainframe-era time-sharing.
  - Expensive, specialized hardware shared across customers.
  - Example: CoreWeave.

- **Global phone tracking**
  - Companies like *FirstWAP* claim to track phones worldwide.
  - No malware required; uses telecom and registration data.

- **Maltego**
  - OSINT tool correlating people, companies, domains, and infrastructure.

**Episode Link:** https://www.grc.com/sn/sn-1052.htm

---

## ☁️ The Cloud Gambit

**Show:** The Cloud Gambit  
**Link:** https://packetpushers.net/podcast/the-cloud-gambit/  
**About:** Strategy, innovation, and cloud-native thinking.

**Key Ideas**
- **Media purpose**
  - Books → learning
  - TV → entertainment
  - Social media → performance
- **Innovation**
  - Happens at intersections of disciplines.
  - Requires theory + practice.
- **Mindset**
  - Focus on solving problems, not just completing tasks.
  - Teaching others deepens understanding.
  - Always understand *why* systems work, not just *how*.

---

## 🌐 The Hedge (with Russ White)

**Show:** The Hedge  
**Link:** https://rule11.tech/the-hedge/  
**About:** Technology, networking, and engineering culture.

**Core Engineering Framework**
- Define:
  - What you need
  - Where it must operate
  - Required interfaces
  - Acceptable risk levels
- Categorize:
  - **Knowns**
  - **Unknowns**
  - **Potential problems**

**Key Insight**
- Good engineering starts with clear constraints and risk awareness.
- Culture and thinking matter as much as technical correctness.

---

## ✅ Cross-Podcast Themes

| Area | Insight |
|-----|--------|
| Intrusions | Phishing, remote access, and vulnerabilities still dominate |
| DNS | Old problems can return due to weak implementations |
| AI | New capabilities require new threat models |
| Cloud | AI compute mirrors historical centralized models |
| Learning | Deep understanding beats shallow consumption |
| Engineering | Explicitly define requirements and risks |

---