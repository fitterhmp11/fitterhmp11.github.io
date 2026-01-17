---
title: "Technical Planning Checklist for Network Engineer"
date: 2026-01-16T021:30:00
tags:
  - concepts
  - technical planning
categories: 
 - networking
draft: false
url: "tech-planning-checklist"
summary: "Technical Planning Checklist for Network Engineer"
description: "Technical Planning Checklist for Network Engineer"
---

# Introduction

This post is based on conversation with ChatGPT exploring elements of Reasoning and Argument, and how they relate to Technical Planning, with focus on network engineer.

# Network Engineer Reasoning & Technical Planning Checklist
*A repeatable, practical checklist for everyday network work (SMB to enterprise)*

---

## Part A — Reasoning Discipline (How to Think)

Use this section to **structure your thinking and communication** before and during any technical work.

### ☐ 1. Clause Clarity (Cause & Effect)
- ☐ Can I clearly state the situation in simple cause/effect terms?
- ☐ Can I say: *If X happens, then Y occurs*?
- ☐ Can I say: *Because X exists, Y is a risk or limitation*?
- ☐ Are my statements specific, not vague?

> Example:  
> ☐ *If the uplink is 1G, then aggregate throughput is limited.*

---

### ☐ 2. Argument Structure (Justification)
- ☐ Do I have at least one clear reason for every major decision?
- ☐ Are my reasons factual (not opinions)?
- ☐ Can I explain my decision without mentioning a product name?

> Example:  
> ☐ *These switches must be replaced because they are unsupported and increase operational risk.*

---

### ☐ 3. Syllogism Check (Logical Soundness)
- ☐ Can my reasoning be reduced to:
  - ☐ A general rule  
  - ☐ A specific case  
  - ☐ A logical conclusion
- ☐ Are there any hidden assumptions I need to validate?

> Example:  
> ☐ *All unsupported gear increases risk → these switches are unsupported → risk is increased.*

---

### ☐ 4. Logical Flow (Order of Explanation)
- ☐ Did I state the problem before proposing a solution?
- ☐ Did I explain constraints before design choices?
- ☐ Did I present risks before final recommendations?
- ☐ Would a non-technical stakeholder follow my explanation?

---

### ☐ 5. Rhetoric (Audience Fit)
- ☐ Am I matching my explanation to the audience (engineer vs manager)?
- ☐ Am I explaining tradeoffs clearly?
- ☐ Am I avoiding unnecessary jargon?
- ☐ Does my explanation build confidence, not confusion?

---

## Part B — Technical Planning Checklist (What to Plan)

Use this section for **every design, replacement, or major change**.

---

### ☐ 6. Requirement Definition — *What must this do?*
- ☐ One-sentence description of the problem to solve
- ☐ Clear success criteria (what “done” looks like)
- ☐ Business or operational driver identified

---

### ☐ 7. Environment — *Where does this live?*
- ☐ Existing topology understood (MDF/IDF, WAN, cloud)
- ☐ IP addressing and VLAN structure documented
- ☐ Legacy equipment identified
- ☐ Power, cooling, rack space confirmed
- ☐ Staffing and operational maturity considered

---

### ☐ 8. Interfaces — *What does it connect to?*
- ☐ Uplinks (speed, media, redundancy)
- ☐ VLANs / SVIs / VRFs
- ☐ DHCP / DNS / RADIUS
- ☐ Monitoring, logging, and alerting systems
- ☐ APIs or automation dependencies

---

### ☐ 9. Knowns — *What information is reliable?*
- ☐ Port counts and device inventory
- ☐ PoE devices and power needs
- ☐ Traffic patterns and peak usage
- ☐ Existing configurations backed up
- ☐ Diagrams and documentation reviewed

---

### ☐ 10. Unknowns — *What must be discovered?*
- ☐ PoE budget under peak load confirmed
- ☐ Uplink media and optics compatibility verified
- ☐ Feature usage validated (ACLs, QoS, stacking)
- ☐ Licensing requirements understood
- ☐ Interoperability tested or confirmed

> ☐ Each unknown has a plan to resolve it

---

### ☐ 11. Risks — *What can go wrong?*
- ☐ Single points of failure identified
- ☐ Cutover downtime risk assessed
- ☐ Firmware or software risks identified
- ☐ Vendor or licensing risks understood
- ☐ Risks ranked by likelihood and impact

---

### ☐ 12. Pre-Mortem — *How could this fail?*
- ☐ Assume failure 30 days post-deployment
- ☐ Identify likely causes (DHCP, STP, auth, routing)
- ☐ Define mitigations for each failure mode
- ☐ Monitoring alerts defined for early detection

---

### ☐ 13. Solution Design — *What fits these constraints?*
- ☐ Solution meets all requirements
- ☐ Design fits the existing environment
- ☐ Risks are mitigated or accepted knowingly
- ☐ Complexity is appropriate for team skill level
- ☐ Total cost of ownership considered

---

### ☐ 14. Verification — *How do we know it worked?*
- ☐ Connectivity tests defined
- ☐ Authentication and security tested
- ☐ Performance validated
- ☐ Monitoring confirms expected behavior
- ☐ Stakeholder acceptance criteria met

---

### ☐ 15. Rollback — *How do we undo this safely?*
- ☐ Previous configs backed up and accessible
- ☐ Old hardware or virtual configs staged
- ☐ Clear rollback steps written
- ☐ Rollback tested or rehearsed
- ☐ Communication plan ready if rollback occurs

---

## Part C — Professional Discipline (After the Change)

---

### ☐ 16. Documentation & Closure
- ☐ Diagrams updated
- ☐ Inventory and IPAM updated
- ☐ Configs committed to version control
- ☐ Lessons learned recorded
- ☐ Support staff informed of changes

---

## One-Sentence Rule to Remember

> **If you can’t explain why you’re doing something, you haven’t planned it yet.**

---

## How to Use This Checklist
- Use it **before** design work begins
- Use it **during** implementation planning
- Use it **after** deployment for verification
- Scale it up or down depending on project size

This checklist is intentionally lightweight —  
formal enough to prevent mistakes, simple enough for SMB reality.