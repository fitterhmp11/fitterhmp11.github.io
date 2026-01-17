---
title: "Packets Through a Switch Forwarding Engine"
date: 2026-01-16T01:00:00
tags:
  - concepts
  - podcasts
categories: 
 - Networking
draft: false
url: "packets-through-switch"
summary: "Packets Through a Switch Forwarding Engine"
description: "Packets Through a Switch Forwarding Engine - notes from Packet Pusher Podcast"
---

# Introduction


Below are my notes after listening to Packet Pusher podcast, then asked AI to organize, clarify and fact-check my notes.

https://packetpushers.net/podcasts/heavy-networking/hn724-how-packets-move-through-a-network-device/

⸻

How Packets Move Through a Modern Switch / Router

(Organized + Fact-Checked Notes)

1. High-Level Hardware Architecture

Control Plane vs Data Plane

Most modern switches and routers are built around two very different processing worlds:

Control Plane (CPU)
	•	Typically an x86 or ARM CPU (small heatsink).
	•	Runs:
	•	Network OS (Linux-based)
	•	Control protocols: BGP, OSPF, STP, LACP
	•	Management: CLI, API, SNMP, telemetry
	•	Not involved in per-packet forwarding under normal operation.

Data Plane (Forwarding Engine / ASIC)
	•	Large heatsink.
	•	Usually one large piece of silicon that handles most or all ports.
	•	Responsible for line-rate packet forwarding.
	•	This is where packets actually move.

✔️ Clarification:
Even modular chassis systems often still use a centralized or distributed ASIC-based forwarding model.

⸻

2. What “ASIC” Really Means

You’re right: ASIC is an overloaded term.

ASIC (Application-Specific Integrated Circuit)
	•	A custom-designed chip optimized for packet processing.
	•	Extremely fast, power-efficient.
	•	Traditionally not changeable after manufacture.

Forwarding Engine
	•	A broader term:
	•	Could be a fixed ASIC
	•	A programmable ASIC
	•	An FPGA-based design
	•	Designed to get packets in → lookup → out as fast as possible.

FPGA (Field-Programmable Gate Array)
	•	Can be reprogrammed after deployment.
	•	Slower and more power-hungry than ASICs.
	•	Often used for:
	•	Prototyping
	•	Specialized features
	•	Niche or low-volume platforms

✔️ Reality today:
Most modern merchant silicon (e.g., Broadcom Trident/Tomahawk) is ASIC-based but partially programmable, allowing vendors to customize packet pipelines.

⸻

3. State Inside a Network Device

Your three-state breakdown is excellent. Here it is refined:

Configuration State
	•	What the operator configured.
	•	Example:
	•	Interfaces
	•	VLANs
	•	BGP neighbors
	•	Stored in:
	•	Running config
	•	Candidate config (depending on OS)

⸻

Operational State
	•	What is actually running right now.
	•	Examples:
	•	BGP session is Established
	•	OSPF adjacency is Full
	•	Lives in the control plane.

⸻

Forwarding State
	•	What the hardware uses to forward packets.
	•	Examples:
	•	MAC address table
	•	IP routing table
	•	MPLS labels
	•	Lives in the data plane (ASIC memory).

⸻

4. RIB vs FIB (Very Important Distinction)

RIB (Routing Information Base)
	•	Control-plane table.
	•	Built from:
	•	BGP
	•	OSPF
	•	Static routes
	•	Can contain many possible routes to the same destination.
	•	Runs on the CPU.

⸻

FIB (Forwarding Information Base)
	•	Data-plane table.
	•	Contains only the best routes selected from the RIB.
	•	Optimized for speed, not flexibility.
	•	Installed directly into the ASIC.

✔️ Key idea:

Routes are calculated in software (RIB) but enforced in hardware (FIB).

⸻

5. Why FIB Lookups Must Be Extremely Fast
	•	Packets arrive back-to-back at line rate.
	•	There is no time to consult the CPU.
	•	The forwarding decision must be:
	•	Deterministic
	•	Constant time
	•	Per packet

This is why specialized memory exists.

⸻

6. CAM and TCAM Explained (Clarified)

CAM (Content Addressable Memory)
	•	Lookup is based on content, not address.
	•	Entire lookup completes in one clock cycle, regardless of table size.
	•	Best for exact matches.

Used for:
	•	MAC address tables (Layer 2)

✔️ Correction:
CAM is not limited to L2 by design, but exact-match requirements made it historically ideal for L2.

⸻

TCAM (Ternary CAM)
	•	Supports 0, 1, or “don’t care” bits.
	•	Allows:
	•	Prefix matching
	•	Longest Prefix Match (LPM)

Used for:
	•	IP routing (L3)
	•	ACLs
	•	QoS classification

✔️ This is the big breakthrough:
TCAM made line-rate L3 forwarding possible, eliminating the old “L3 is slow” problem.

⸻

7. Hardware vs Software FIB
	•	Hardware FIB
	•	Stored in CAM / TCAM
	•	Line-rate forwarding
	•	Software FIB
	•	Exists in the OS (CPU)
	•	Used when:
	•	Feature unsupported in hardware
	•	Table overflow
	•	Debugging or slow paths

✔️ Rule of thumb:

If it’s in hardware, it’s fast.
If it hits the CPU, performance collapses.

⸻

8. Merchant Silicon Reality

You’re spot on here.
	•	Many vendors use the same chipmakers:
	•	Broadcom
	•	Intel (Tofino)
	•	Marvell
	•	Core capabilities often come from the chip:
	•	Table sizes
	•	Port speeds
	•	Feature limits

Vendor differentiation comes from:
	•	How they program the pipeline
	•	Control plane software
	•	Feature exposure and tuning

⸻

9. What “Line Rate” Really Means

This is one of the most misunderstood concepts.

Packet Size Matters
	•	Smaller packets = more packets per second
	•	More packets per second = less time per packet

Example:
	•	64-byte packets at 100G = ~148 million packets/sec

⸻

Link Speed Matters
	•	Faster links = shorter gaps between packets
	•	10G → 100G → 400G → 800G
	•	Each jump reduces available processing time

⸻

Clock Cycles and Physics
	•	1 GHz clock ≈ 1 nanosecond per cycle
	•	CAM/TCAM lookup typically takes one clock cycle
	•	That becomes a hard performance limit

✔️ Important insight:
No matter how fast CPUs get, physics + memory access time ultimately cap forwarding speed.

⸻

10. Clean Mental Model (TL;DR)

Control plane thinks.
Data plane acts.

	1.	CPU runs protocols and decides what should happen
	2.	Best decisions go into the FIB
	3.	ASIC forwards packets at line rate using CAM/TCAM
	4.	Smaller packets + faster links = harder problem
	5.	Hardware forwarding is the only way this works at scale

⸻
