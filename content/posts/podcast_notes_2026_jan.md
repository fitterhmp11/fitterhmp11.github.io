---
title: "Podcast Notes: January 2026"
date: 2026-02-01T01:00:00
tags:
  - podcasts

categories:
  - Networking
  - Security

draft: false
url: "podcasts-01-2026"
summary: "Notes from Podcasts listened to January 2026"
description: "Notes from Podcasts listened to January 2026"

---

# Below are some interesting points I found from podcasts listened to during January 2026

---

## **Security Now! #1056**

**Key Points:**

* **Home Depot API Leak:** A researcher found a Home Depot API key in a public Git repo. Notably, the company did not respond to the disclosure until Ars Technica reached out, highlighting a gap in their incident response pipeline.
* **Let's Encrypt Scale:** The service now renews 10 million certificates daily. This raises concerns about a single point of failure for the global internet's encryption infrastructure.
* **ICMP Echo Principles:** A reminder of a core internet principle: any IP-enabled device should respond to ICMP Echo requests to maintain network visibility and diagnostic capabilities.

**Reference:** [Security Now! #1056](https://www.google.com/search?q=https://www.grc.com/sn/sn-1056.htm)

---

## **Heavy Networking #724**

**Key Points:**

* **Switch Architecture:** Modern routers/switches typically feature a standard x86 CPU for the Control Plane (smaller heat sink) and a specialized Forwarding Engine or ASIC (large heat sink) for the Data Plane.
* **ASIC vs. FPGA:** While "ASIC" is used ambiguously by vendors, it generally refers to a fixed-design chip for high-speed frame processing. In contrast, an FPGA can be reprogrammed or changed after manufacturing.
* **Understanding Network States:**
* **Configuration State:** The desired state (Running Config).
* **Operational State (RIB):** The protocol-derived state (BGP/OSPF tables).
* **Forwarding State (FIB):** The hardware-level table (L2/L3) optimized for line-rate speed.


* **CAM vs. TCAM:** * **CAM (Content Addressable Memory):** Used for L2; requires an exact match but provides a result in a single clock cycle.
* **TCAM (Ternary CAM):** Enables "Longest Prefix Match" for L3 subnets, allowing L3 switching to reach hardware speeds similar to L2.


* **Line Speed Realities:** As packet sizes decrease or media speeds (100G/800G) increase, the time the switch has to process a packet shrinks. High-speed lookups are limited by the CPU's clock cycle (e.g., 1ns for 1GHz).

**Reference:** [Heavy Networking #724](https://www.google.com/search?q=https://packetpushers.net/podcasts/heavy-networking/hn724/)

---

## **Heavy Networking: EVPN All Things**

**Key Points:**

* **Learning Path:** The recommended progression for modern fabrics is to master eBGP first, then transition into VXLAN and EVPN.
* **Multi-tenancy:** The primary functional advantage of this architecture is the ability to provide robust L3 segmentation for multi-tenant environments.

**Reference:** [Heavy Networking: EVPN/VXLAN Series](https://www.google.com/search?q=https://packetpushers.net/series/evpn-vxlan/)

---

## **Security Now! #1059**

**Key Points:**

* **Automated Code Signing:** The WWW Consortium is moving toward an automated process for code signing which will likely require a subscription model with Certificate Authorities.
* **PyPI Security:** To combat phishing, the PyPI repository is adding an extra factor to TOTP codes for developer accounts.
* **BitLocker Performance:** Microsoft has noted that BitLocker can bottleneck high-speed NVMe drives because the drive speed now exceeds the encryption processing capacity.

**Reference:** [Security Now! #1059](https://www.google.com/search?q=https://www.grc.com/sn/sn-1059.htm)

---

## **Packet Protectors #093**

**Key Points:**

* **2026 Security Priorities:** The focus remains on the basics—log correlation in SIEM and regular firewall policy reviews.
* **Emerging Risks:** There is growing concern regarding LLM-driven proprietary data leakage and the shortening "time-to-exploit" window for vendor bugs.
* **Enterprise Browsers:** A shift toward using enterprise-grade browsers to secure content directly on the endpoint.
* **SAML Everywhere:** Some organizations are moving away from traditional VPNs in favor of public resources protected strictly by SAML for authentication and authorization.

**Reference:** [Packet Protectors #093](https://www.google.com/search?q=https://packetpushers.net/podcasts/packet-protector/pp093/)

---

## **The Hedge #292**

**Key Points:**

* **Data Center Roots:** Modern fiber paths often follow historical train tracks because the rights-of-way were established during the telco era.
* **Redundancy vs. Resilience:** It is critical to distinguish between having "extra" parts (redundancy) and the system's ability to recover from failure (resilience).
* **Power & SMRs:** Data center racks can average 250kW. To meet these massive power demands, there is a push for Small Modular Reactors (SMRs)—factory-made nuclear reactors providing ~300MW.

**Reference:** [The Hedge #292](https://www.google.com/search?q=https://rule11.tech/podcasts/the-hedge/)

---

## **The Hedge #265**

**Key Points:**

* **Out-of-Band (OOB) Networks:** OOBM should not be reserved only for disasters. It should be the primary path for all management, automation, and observability traffic.
* **The Highway Analogy:** If the production network is the highway, the OOBM is the system of maintained tunnels and drainage beneath it.
* **Security via Isolation:** Segmentation protects production bandwidth and ensures that the only access to infrastructure is through the managed OOB path.

**Reference:** [The Hedge #265](https://rule11.tech/hedge-265/)

---

## **Security Now! #1060**

**Key Points:**

* **Code Signing Costs:** The shift in signing processes could cost independent developers upwards of $1,000 per year.
* **California DROP:** A new CA platform allows residents to submit PII deletion requests that data brokers are legally required to follow.
* **How Code Signing Works:** Unlike TLS, code signing proves the identity of the signer and ensures the code hasn't been altered since the signature was applied. Using a Time Stamping Authority (TSA) verifies the signature was valid at the specific time of application.

**Reference:** [Security Now! #1060](https://www.google.com/search?q=https://www.grc.com/sn/sn-1060.htm)

---

## **Packet Protectors #746 (OSPF Filtering)**

**Key Points:**

* **OSPF Synchronization:** Because OSPF is link-state, every router in an area must have an identical Link State Database (LSDB) to perform SPF calculations.
* **Filtering Limitations:** You cannot filter routes within an area without breaking the LSDB sync. To filter what enters the routing table without breaking the database, you must use a `distribute-list` locally.

**Reference:** [Packet Protectors #746](https://www.google.com/search?q=https://packetpushers.net/podcasts/heavy-networking/hn746/)

