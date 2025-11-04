---
title: "Podcast Notes - September 2025"
date: 2025-09-30T18:00:00
tags:
  - concepts
  - networking
  - cybersecurity
  - automation
  - python
categories: 
 - Networking
draft: false
url: "podcast-notes-september-2025"
summary: "Monthly listening notes covering identity and authentication protocols, network automation strategy (build vs buy), latency troubleshooting across all OSI layers, and security vulnerabilities in supply chain packages."
description: "Curated notes from September 2025 podcast episodes including Security Now!, Heavy Networking, Packet Protectors, and Tech Chats covering SAML vs OAuth vs OIDC, automation reality checks, and enterprise security challenges."
---

# Podcast Notes - September 2025

Monthly collection of key takeaways and insights from networking and security podcasts.

---

## Packet Protectors 073: Identify Yourself

**Podcast:** [Packet Protectors - Packet Pushers Network](https://packetpushers.net/series/packet-protectors/)

### Authentication and Authorization Protocols

**SAML (Security Assertion Markup Language)**
- XML-driven authentication protocol
- Tokens stored in browser
- Common in enterprise SSO (Single Sign-On) deployments

**OAuth**
- **Authorization protocol, NOT authentication**
- Uses JSON tokens
- Backend communication behind the scenes, not stored in browser
- Grants access to resources without sharing credentials

**OIDC (OpenID Connect)**
- Builds on OAuth 2.0
- Provides authentication layer (OAuth doesn't)
- Powers social logins with authentication
- Common for "Sign in with Google/Microsoft/GitHub" functionality

**Common Assumption**
- All three protocols assume user is already logged in
- Focus is on passing authentication/authorization tokens between systems

### FIDO Alliance and Passkeys

**FIDO2 Standard**
- Modern authentication framework from FIDO Alliance
- Eliminates passwords in favor of cryptographic keys

**Passkey Architecture**
- Can chain passkey to a hardware security token
- Uses cryptographic authentication with public/private key pairs
- Passkeys designed to be passed using WebAuthn API
- More secure than traditional passwords (resistant to phishing)

---

## Security Now! 1042: Shiny Hunters and Certificate Authority Trust

**Podcast:** [Security Now! with Steve Gibson](https://www.grc.com/securitynow.htm)

### Threat Actor Spotlight

**Shiny Hunters Group**
- Highly successful phishing group
- Known for large-scale data breaches through social engineering

### Certificate Authority Incident

**Mistaken SSL Certificates**
- Certificates mistakenly issued by Certificate Authority
- Incident detected through certificate transparency logs
- Demonstrates the power (and risk) of root CA trust
- Highlights importance of CT (Certificate Transparency) monitoring

### Critical Problem-Solving Mindset

**"I Don't Know" is a Great and Important Phrase**

**"I don't know" is a great and important phrase, because it is the first step in finding out the answer. Guessing at an answer may introduce pride into the equation, trying to prove yourself right. Rather than proving something right, finding the real problem is what is most important.**

- Admitting knowledge gaps enables learning
- Guessing introduces ego and defensiveness
- Focus on finding truth, not being right
- Critical skill for troubleshooting and incident response

---

## Heavy Networking 791: Should You Build or Buy Your Automation Platform?

**Podcast:** [Heavy Networking - Packet Pushers Network](https://packetpushers.net/series/heavy-networking/)

### Third-Party Automation Platforms

**Gluware Automation Platform**
- Third-party paid automation platform
- Alternative if you don't have time or resources to build custom Ansible/Python automation
- Explore third-party options rather than abandoning automation entirely

### Out-of-Band Management

**Opengear - Cellular Out-of-Band Networking**
- Provides cellular-based OOB (out-of-band) access
- Critical for remote sites and disaster recovery
- Maintains management access when primary network fails

### Workflow Design Principles

**Automation Workflows Defined by Device or SKU**
- Device-specific automation templates
- NOTE:**SKU-based configuration management**
- Reduces complexity by standardizing per platform

---

## Tech Chats 056: Network Automation Reality Check

**Podcast:** [Tech Chats - Packet Pushers Network](https://packetpushers.net/series/tech-chats/)

**This particular episode was good and can be re-listened to for deeper insights.**

### Network Importance by Business Type

**Core Product vs Supporting Infrastructure**

**Hyperscalers and ISPs (AWS, Azure, Telcos):**
- Network IS their core product
- Heavy investment in automation justified
- Network directly generates revenue

**Most Other Enterprises:**
- Have some other core product (manufacturing, retail, finance)
- Network viewed as necessary element to do business
- Supporting infrastructure, not revenue generator
- Automation investment must be proportional

### Enterprise Buying Behavior

**Why Enterprises Buy Tools**
- Solve a specific problem
- **Distance themselves from responsibility to fix it**
- Shift accountability to vendor
- Alternative: Hire consultant to implement/maintain

### Tooling Landscape Considerations

**Declarative vs Imperative Approaches**

**Network Devices:**
- Usually use **declarative** configuration (desired state)
- Define what you want, tool figures out how to get there
- Examples: Ansible, Terraform for network

**Cloud Infrastructure:**
- Usually use **imperative** configuration (step-by-step)
- Define exact steps to achieve result
- Examples: AWS CLI, Azure scripts

### Critical Automation Principles

**Each automation solution must be determined by what device and network environment it operates in.**

- No one-size-fits-all automation solution
- Platform-specific approaches required
- Vendor APIs and capabilities dictate tooling choices

**Don't try to automate brownfield networks. Build new system and scale from there.**

- Brownfield automation is complex and risky
- Start with greenfield deployments
- Prove automation value, then expand
- Migration strategy beats big-bang conversion

**Planning Over Immediate Solutions**
- Don't have to solve the problem today
- Plan to solve the problem in the future
- Build capability incrementally
- Avoid rushed automation that creates technical debt

### Automation Implementation Challenges

**Ways to Automate Are Problematic**

**SSH/CLI Automation:**
- SSH has higher priority in device CPU scheduling
- Steals CPU cycles from forwarding plane
- Can impact production traffic during bulk operations

**API Calls:**
- Suffer from vendor limitations
- Inconsistent implementations across vendors
- Rate limiting and timeout issues
- Some vendors have incomplete API coverage

**Screen Scraping CLI:**
- Parsing CLI output is difficult and fragile
- Vendor output formats change between versions
- Error handling is complex
- Brittle and maintenance-intensive

**Vendor Standardization Needed**
- Vendors need to standardize output of API calls
- Industry-wide data models (YANG, OpenConfig) help
- Inconsistency is biggest automation barrier

### Open Source Sustainability

**Open Source Run by Enterprises**
- Open source projects backed by companies
- Will always be looking at how they can eventually make money
- Understand business model behind "free" tools
- Plan for potential licensing changes or feature paywalls

---

## Heavy Networking 795: Adventures in Latency

**Podcast:** [Heavy Networking - Packet Pushers Network](https://packetpushers.net/series/heavy-networking/)

### Understanding Latency Sources

**Latency Can Be Found in OSI Layers 1-7**

**Latency can be found in layers 1-7 of the OSI model. Network latency is a subset of total latency—it can be the host, not the network.**

- Don't assume latency is always network-related
- Application layer (L7) can introduce significant delays
- Server/host processing time often exceeds network transit time
- Database queries, API calls, disk I/O all contribute
- Comprehensive troubleshooting requires examining entire stack

### Long Fat Networks (LFN)

**LFN Characteristics**
- Connect huge educational/research data centers across the world
- High bandwidth × high latency = large bandwidth-delay product
- Requires TCP window scaling and tuning
- Common in research networks (Internet2, ESnet)
- Special considerations for protocol optimization

---

## Security Now! 1044: Chrome Zero-Day and Supply Chain Risks

**Podcast:** [Security Now! with Steve Gibson](https://www.grc.com/securitynow.htm)

### Chrome Security Response

**Rapid Zero-Day Patch**
- Chrome fixed actively exploited zero-day vulnerability in 1 day
- Demonstrates capability for rapid security response when needed

**Strategic Vulnerability Disclosure**
- Also updated other critical vulnerabilities known for a while
- Previously waited to patch, perhaps observing to see if exploited
- Strategic timing of patches balances security vs intelligence gathering

### Supply Chain Security

**Malicious NPM and Node Community Packages**

**More NPM/Node community packages are found to be malicious. Segmentation and isolation is key. Software or hardware that makes use of vulnerable packages can become malicious without knowledge of the operators.**

- Growing number of malicious packages in NPM ecosystem
- Software supply chain attacks increasing
- **Segmentation and isolation is critical defense**
- Dependencies can introduce vulnerabilities unknowingly
- Transitive dependencies (dependencies of dependencies) expand attack surface
- Software or hardware using vulnerable packages becomes attack vector
- Operators may be unaware their systems are compromised
- Defense strategies:
  - Network segmentation to limit blast radius
  - Isolation of critical systems
  - Dependency scanning and vulnerability management
  - Software Bill of Materials (SBOM) tracking
  - Limit use of unnecessary packages

---

## Resources

- **Security Now!** - Weekly security podcast by Steve Gibson and Leo Laporte: https://www.grc.com/securitynow.htm
- **Packet Pushers Network** - Networking podcast network covering design, automation, and security: https://packetpushers.net/
- **Heavy Networking** - Deep-dive networking topics: https://packetpushers.net/series/heavy-networking/
- **Packet Protectors** - Cybersecurity and network security: https://packetpushers.net/series/packet-protectors/
- **Tech Chats** - Networking career and technology discussions: https://packetpushers.net/series/tech-chats/

---

*These notes represent personal takeaways and key concepts from podcast episodes. For complete context and details, listen to the full episodes at the links above.*