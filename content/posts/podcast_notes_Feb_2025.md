---
title: "Podcast Notes: February 2025"
date: 2025-03-01T01:00:00
tags:
  - podcasts
categories: 


draft: false
url: "podcasts-02-2025"
summary: "Notes from Podcasts listened to February 2025"
description: "Notes from Podcasts listened to February 2025"
---
# Below are some  interesting points I found from podcasts listened to during February 2025

# February 2025 Podcast Notes

## Security Now! 1011

Deepseek ClickHouse DB server found exposed. ClickHouse is an open-source database for real-time data processing and big data analytics. The rapid expansion of AI solutions shows that security best practices may be overlooked. Rapid adoption means rushed, overlooked security.

- **Query time compute** - AI spends analyzing before replying to reduce hallucinations.
- **AI jailbreaking techniques** - Ways to bypass AI restrictions.
- **Guardrails** - Implementations in AI to prevent unauthorized use.
- **Deepseek R1** shows significant vulnerability to jailbreak techniques.

=> >"Trainable, conversational, ethically naive knowledge extraction systems…tricking them to share dangerous knowledge is not that difficult."

Reference: [Security Now! Podcast](https://twit.tv/shows/security-now).

---

## NAN084: From GitNops Zero to Hero

- Example of **GitHub flow** for network engineers:
  - Router configuration in a GitHub repo.
  - Scott logs in and **forks** the repo (creates a copy connected to the original, so if Eric makes changes, Scott can pull updates).
  - Scott makes a change to the router configuration (e.g., VLAN ID change).
  - Scott submits a **pull request** to the original repo.
  - Eric reviews the changes (GitHub issue) and manages the update.
- Beneficial for network changes due to version control and peer review.

**Key technologies network engineers need to know:**  
=> Git (inside and out), Python, Linux, IaC (Terraform).

=> Great episode to review again


Reference: [Network Automation Nerds Podcast](https://packetpushers.net/podcasts/network-automation-nerds/nan084-from-gitnops-zero-to-hero/).

---

## Security Now! 1012

- **SparkCat** - Malicious apps in the Google Play Store scanning pictures on phones, looking for cryptocurrency keys or other sensitive information stored as screenshots.
  - **Advice:** Install as few apps as possible; be cautious of third-party apps.

- **UK law forcing backdoors** - Proposed legislation weakens end-to-end encryption, prompting Apple to consider removing **Advanced Data Protection (ADP)**.

- **CISA Five Eyes Edge Security Guidance:**  
  [Read the full guidance here](https://www.cisa.gov/resources-tools/resources/guidance-and-strategies-protect-network-edge-devices).

- **Sysinternals vulnerability** - Unpatched flaw allows access to local DLL files.  
  - **Windows Sysinternals** is a collection of 70 freeware utilities from Microsoft for **monitoring, managing, diagnosing, and troubleshooting Windows systems**.

Reference: [Security Now! Podcast](https://twit.tv/shows/security-now).

---

## HN769: CI/CD Pipelines and Network Automation

### Benefits for Network Automation:
- **Scale** (more devices)
- **Complexity** (harder for humans to configure)
- **Frequency of changes** (is more often better?)
- **Accuracy** (ensuring changes work as expected)

> "Arista says we're just going to call it **continuous integration**. They don't even use CI/CD separately because they're married together."

### YAML in Network Automation:
- YAML defines the **desired network config** in a human-readable format.
- Linting checks YAML for correct formatting.
- **Ansible** converts YAML into device-specific syntax using Python.

Most network automation does not apply small config snippets but replaces the entire device config:
- **Source of truth:** YAML file.
- **Device states:** Previous config → New config.  
  - If updating in snippets, there's an **uncertain third state** where you don’t know if the config is working.

=> Great episode to review again

Reference: [Heavy Networking Podcast](https://packetpushers.net/podcasts/heavy-networking/).

---

## Security Now! 1013

- **UK demand for backdoor access** in encryption.
- **Ransomhub** - Most active ransomware gang of 2024.
- **Security research on Google Web Store:**  
  - Google takes a **minimal effort** approach to security for web apps.
  - Major concern because **Google Chrome** powers many web apps.

Reference: [Security Now! Podcast](https://twit.tv/shows/security-now).
