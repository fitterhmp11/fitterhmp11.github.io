---
title: "Network Design Guide: Aruba CX & Ruckus ICX Core/Aggregation to Access"
date: 2025-10-27T10:00:00
tags:
  - networking
  - design
  - concepts
  - aruba
  - ruckus
categories: 
 - Networking
draft: false
url: "network-design-aruba-ruckus-switching-stacks"
summary: "Comprehensive reference guide for designing medium enterprise campus networks with Aruba CX and Ruckus ICX switches, covering stacking, VSX, MCLAG, and proper sizing methodology."
description: "Technical design guide for network engineers covering switching capacity, fabric architecture, oversubscription ratios, VSX vs MCLAG comparison, and operational best practices for campus network deployments."
---

# Network Design Summary — Aruba CX & Ruckus ICX (Core/Aggregation → Access)

This single-file reference condenses essential design principles into a compact, actionable guide for **medium enterprise campus** deployments (MDF + IDFs, collapsed core options). It includes definitions, sizing math, vendor distinctions (VSX vs stacking vs MCLAG), and clear best-practice checklists for both **Aruba CX** and **Ruckus ICX** environments. The audience is both design engineers and operational staff.

---

## Executive summary — main topics covered

* **Core concepts & definitions:** switching capacity, fabric/backplane, forwarding rate (Mpps), stack links, ISL/VSX, MLAG/MCLAG.
* **Sizing rules and math:** oversubscription (default 20:1), mapping 1G edge ports → required uplink and recommended stack bandwidth.
* **Stacking vs multi-chassis:** trade-offs between VSF/stack (single control plane) and VSX/MLAG (multi-chassis active-active), and where MCLAG fits.
* **Aruba VSX vs Ruckus ICX MCLAG:** accurate comparison of behavior, state sync, L2/L3 handling, and operational differences.
* **Operational monitoring:** what to watch on stack ports vs uplinks; counters and thresholds.
* **Best practices & checklist:** 12 topology best practices plus specific recommendations for stacking, uplink sizing, MDF switch selection criteria.

---

## Quick decision tree (uplink & stack sizing)

```
START: Count total 1G edge ports across all IDFs
  │
  ├─→ ≤200 ports (1-2 small IDFs)
  │     Uplink: 1×10G per IDF (or 2×10G for redundancy)
  │     Stack: 40G aggregate minimum
  │     MDF: Non-blocking ≥20-40G fabric, ≥30-60 Mpps
  │
  ├─→ 200-400 ports (3-4 medium IDFs)
  │     Uplink: 2×10G LAG or 1×40G per IDF
  │     Stack: 80G aggregate minimum (prefer 40G per-hop)
  │     MDF: Non-blocking ≥120-160G fabric, ≥90-240 Mpps
  │     Consider: VSX/MLAG for distribution pair
  │
  ├─→ 400-800 ports (5-8 IDFs)
  │     Uplink: 1×40G or 2×25G per IDF
  │     Stack: 160G+ aggregate (40G+ per-hop required)
  │     MDF: Non-blocking ≥240-320G fabric, ≥360-480 Mpps
  │     Strongly recommend: VSX/MLAG distribution pair
  │
  └─→ >800 ports (9+ IDFs)
        Uplink: 1-2×40G or 2-4×25G per IDF
        Stack: 320G+ aggregate OR split into multiple stacks
        MDF: Consider chassis or high-density modular switches
        Required: VSX/MLAG distribution, possibly spine-leaf architecture
```

**Rule of thumb:** Stack bandwidth should be ≥2× aggregate uplink bandwidth for headroom.

---

## Short, precise definitions (engineer-focused)

* **Switching capacity (Gbps)** — Theoretical aggregate throughput of the switch fabric/ASIC matrix (often advertised as full-duplex aggregate). Use as a coarse upper-bound indicator, not a guarantee of real-world throughput.

* **Fabric** — The ASIC-level switching matrix or crossbar architecture that realizes packet forwarding between ports. This is the logical/electronic switching mechanism inside the device.

* **Backplane** — The physical interconnect (PCB traces, connectors, optical paths) that carries data between line cards, modules, or stack members. In modular chassis switches, this is a literal physical backplane. In fixed switches, it's integrated into the circuit board. In stacks, the "backplane" is effectively the stacking cables/links.

* **Fabric vs Backplane clarification:** Modern fixed switches integrate both—the ASIC fabric and physical interconnects are on the same board. In modular chassis, the backplane is the physical infrastructure, and each line card contains fabric ASICs. For stacks, the stacking cables become the physical "backplane" while each member has its own local fabric. **Key point:** Both must have adequate capacity; the lower of the two becomes your practical limit.

* **Forwarding rate (Mpps)** — Packets per second the device can process at small-packet sizes (typically 64-byte); critical for small-packet workloads (VoIP, telemetry, gaming, IoT). Calculated as: **(Line rate in bps) / (64 bytes × 8 bits/byte + 20 bytes overhead)** ≈ **1.488 Mpps per Gbps**.

* **Stack port / stacking link** — Physical inter-switch links (dedicated stacking ports or standard Ethernet ports configured for stacking) forming a logical stack. These links are the practical bottleneck for cross-member traffic and determine real-world stack performance.

* **Uplink** — Link from access/IDF switch to MDF/core (e.g., 10G, 25G, 40G, 100G); typically aggregated via LAG (Link Aggregation Group / Port-Channel).

* **Oversubscription ratio** — Ratio of total edge port capacity to uplink capacity (edge ports : uplink). **20:1** is a common office default, meaning 20 Gbps of edge capacity shares 1 Gbps of uplink.

* **VSF/Stack (virtual chassis)** — Multiple physical switches operating as one logical device with a single control plane, single management IP, and shared configuration. One master/commander switch handles control plane; failure promotes a backup.

* **VSX / MLAG / MCLAG** — Multi-chassis active-active link aggregation where two (or more) independent switches with separate control planes present a single LACP peer to downstream devices. **VSX** (Aruba) provides richer state synchronization including routing protocol state and Active Gateway support. **MCLAG** (ICX/Ruckus and others) provides simpler active-active L2 redundancy with manual configuration mirroring.

---

## Concise analysis of the most important points

1. **Real-world limits are physical links you install.** Vendor "fabric" numbers are useful ceiling indicators, but the installed stacking links and uplinks are the real-world throughput limits. Always size real links first based on aggregate traffic requirements.

2. **Oversubscription governs uplink selection.** For typical office users a **20:1** ratio is a reasonable starting point: 10G uplink → ~200×1G ports; 20G uplink → ~400×1G ports. Adjust for workload: heavy collaboration areas use 10:1, server/storage requires 1-2:1.

3. **Stack bandwidth must match aggregate upstream load.** Rule of thumb: **stack_bw ≥ total IDF uplink aggregate (minimum)**; **recommended = 2× aggregate** for headroom and failure resilience; **3–4× aggregate** for heavy east-west traffic (local server VLANs, peer-to-peer collaboration).

4. **Per-hop bandwidth matters in multi-hop rings.** A ring of 4 switches with 10G stacking links has 40G aggregate bidirectional capacity, but each individual hop is only 10G. Multi-hop flows easily hit per-link limits. Prefer higher per-hop speeds (40G) for larger stacks. **Ring failure warning:** When a ring breaks, all traffic goes one direction—plan capacity accordingly or use dual-ring/mesh topologies for critical deployments.

5. **VSX (Aruba) offers richer L3-aware multi-chassis behavior; ICX MCLAG is simpler.** VSX provides configuration and control-state synchronization and supports Active Gateway (distributed L3 default gateway across both switches)—excellent for performance and hitless failover. MCLAG provides active-active L2 redundancy, requires manual config mirroring (manageable with automation tools like Ansible), and does not synchronize routing protocol state.

6. **Design for failure.** After a single link or node failure, remaining capacity should be ≥ minimum required aggregate. Use dual uplinks, distribute MDF uplinks across stack members, or use MLAG/VSX to avoid central stack traversal and single points of failure.

7. **Monitor and iterate.** Use sFlow/NetFlow and interface counters to validate design assumptions. Thresholds: warn at sustained **>50–60%** utilization of stack links or uplinks; plan capacity upgrades at **>70–80%** sustained utilization.

---

## Actionable sizing tables (quick reference)

**Assumptions:** 1 Gbps ≈ 1.488 Mpps (small-packet wire-rate forwarding). Oversubscription = 20:1.

### A. IDF uplink aggregates (single 10G from each IDF)

| # IDFs | Aggregate (Gbps) | Min fabric | Recommended fabric (2×) | Min forwarding (Mpps) | Rec forwarding (Mpps) |
| -----: | ---------------: | ---------: | ----------------------: | --------------------: | --------------------: |
|      1 |               10 |         10 |                      20 |                  14.9 |                  29.8 |
|      3 |               30 |         30 |                      60 |                  44.6 |                  89.3 |
|      6 |               60 |         60 |                     120 |                  89.3 |                 178.6 |
|      8 |               80 |         80 |                     160 |                 119.0 |                 238.1 |
|     10 |              100 |        100 |                     200 |                 148.8 |                 297.6 |

### B. IDF uplink aggregates (2×10G per IDF = 20G each)

| # IDFs | Aggregate (Gbps) | Min fabric | Recommended (2×) | Min Mpps | Rec Mpps |
| -----: | ---------------: | ---------: | ---------------: | -------: | -------: |
|      1 |               20 |         20 |               40 |     29.8 |     59.5 |
|      3 |               60 |         60 |              120 |     89.3 |    178.6 |
|      6 |              120 |        120 |              240 |    178.6 |    357.1 |
|      8 |              160 |        160 |              320 |    238.1 |    476.2 |
|     10 |              200 |        200 |              400 |    297.6 |    595.2 |

### C. IDF uplink aggregates (single 40G from each IDF)

| # IDFs | Aggregate (Gbps) | Min fabric | Recommended (2×) | Min Mpps | Rec Mpps |
| -----: | ---------------: | ---------: | ---------------: | -------: | -------: |
|      1 |               40 |         40 |               80 |     59.5 |    119.0 |
|      3 |              120 |        120 |              240 |    178.6 |    357.1 |
|      6 |              240 |        240 |              480 |    357.1 |    714.3 |
|      8 |              320 |        320 |              640 |    476.2 |    952.4 |
|     10 |              400 |        400 |              800 |    595.2 |   1190.5 |

**How to use:** Find your number of IDFs and chosen uplink speed, then select an MDF switch with fabric capacity and forwarding rate meeting or exceeding the "Recommended" column values.

---

## Best-practice checklist (12+ focused items for core/agg → access)

1. **Standardize SKUs & templates.** Keep 1–2 access SKUs and 1–2 MDF SKUs per vendor family. Document PoE budgets, uplink types, optics compatibility, and cable requirements in a reference database.

2. **Use 48-port switches as default unless small-site exception.** Better cost per port ($/port) and future growth capacity; reserve 24-port switches for special cases (small IDFs, specialized zones).

3. **Oversubscription rule:** Default **20:1** for general office users; use **10:1** for heavy-collaboration areas (engineering, video editing, design); **1–2:1** for server/storage uplinks.

4. **Stack sizing:** Stack aggregate bandwidth ≥ total IDF uplink aggregate (minimum); recommended = 2× aggregate. Ensure that after a single-link or single-member failure, remaining capacity ≥ minimum required.

5. **Per-hop awareness:** Avoid relying on many 10G hops in a ring topology. Prefer 40G per-hop bandwidth for stacks >4 switches or >200 total ports. Consider mesh or dual-ring for critical paths.

6. **Uplink redundancy:** Always dual-home the IDF stack to MDF (2×10G is common minimum). Distribute uplinks across different stack members to reduce intra-stack traffic and avoid single-member bottlenecks.

7. **Consider MLAG/VSX for heavy uplinks or many IDFs.** If you don't currently run MLAG/VSX, propose it for deployments with high aggregate bandwidth or large IDF counts (>4 IDFs). VSX provides superior L3 behavior and state synchronization compared to traditional MCLAG.

8. **Monitor stack ports & uplinks:** Alert at sustained 50–60% utilization; plan capacity changes when utilization exceeds 70–80% during business hours.

9. **Size for forwarding rate (Mpps):** Verify vendor Mpps specifications meet or exceed your recommended Mpps from the sizing tables. Critical for environments with many small-packet flows (VoIP, IoT sensors, industrial telemetry).

10. **Plan peer-link (ISL) sizing:** For VSX/MLAG deployments, size the peer/ISL link ≥ expected cross-peer traffic volume; recommended **40G minimum** for modern deployments, **100G** for large-scale or EVPN environments.

11. **Balance VLANs and uplinks across stack members** to minimize cross-member flows and stack link utilization. Place uplinks and their corresponding VLANs on the same member when possible.

12. **Power/PoE checks & margins:** Ensure PoE budget versus expected device load (APs, phones, cameras). Verify rack power circuits and UPS capacity. Add 20% margin for growth and efficiency losses.

13. **Document & automate:** Maintain a live database (spreadsheet, Airtable, or DCIM tool) of datasheet extracts including: port counts, PoE budgets, uplink types, stacking capabilities, fabric/Mpps specs, and current pricing. This accelerates quoting and design validation.

---

## Specific Aruba CX & Ruckus ICX design tips (practical)

### Aruba CX (VSF / VSX)

* **VSF/stack**: Easy management with single control plane and unified configuration. Suitable for small to medium stacks (up to vendor-specified member limits, typically 4-10 switches). Ensure stack backplane bandwidth (installed stacking cables) meets or exceeds uplink aggregate requirements.

* **VSX**: Preferred architecture for high-availability aggregation/core or distribution layer. Each switch maintains an independent data plane and control plane; the ISL (Inter-Switch Link) carries only synchronization traffic and cross-peer data flows. VSX supports **Active Gateway** (distributed L3 default gateway on both switches)—excellent for performance and hitless failover during maintenance or failures.

* **Port compatibility:** Newer CX models support SFP+ (10G), SFP28 (25G), and SFP56 (50G) optics with tri-speed auto-negotiation. Check specific SKU datasheets and firmware versions for compatibility. For future-proofing, prefer switches with SFP56 ports if you anticipate 25/50G uplink upgrades.

* **Peer/ISL sizing:** Start with **40G minimum** for ISL (typically 1-2×40G or 4×10G LAG); upgrade to **100G** if expected cross-peer flows are heavy or for large EVPN/VXLAN overlay deployments.

* **Operational tips:** 
  - Verify VSX keepalive link is separate from ISL (dedicated out-of-band management connection recommended)
  - Test failover scenarios before production (cable pull, reload, upgrade)
  - Plan hitless software upgrades using VSX ISSU (In-Service Software Upgrade) capabilities

### Ruckus ICX (Stacking / MCLAG)

* **Stacking (ICX)**: Well-suited for access layer; dedicated stacking modules/ports can provide high aggregate stack bandwidth. Real-world limit is installed stacking links and per-hop bandwidth. Use ring topology where possible for redundancy; consider dual-ring for critical deployments.

* **MCLAG**: Use for dual-homing access switches when you need active-active L2 redundancy but don't require rich L3 state synchronization. MCLAG requires manual configuration mirroring across peer switches (document configurations precisely; use automation tools like Ansible for consistency).

* **Stack bandwidth considerations:** If using stacked ICX with 10G stacking links, aggregate stacking bandwidth may appear adequate on paper, but per-hop 10G limits can create bottlenecks for multi-hop flows. Move to QSFP/40G stacking modules for larger stacks (>4 members or >200 ports).

* **Operational tips:**
  - Monitor per-stack-link interface counters, CRC errors, and output drops
  - Errors on stack ports impact the entire logical stack—address immediately
  - Validate stack member priority and preemption settings to control master election
  - Test stack split scenarios (ring break) to verify failover behavior

---

## Operational checks & monitoring (quick commands/actions)

### Key metrics to monitor:

* **Stack link utilization:** Input/output rates on stacking interfaces
* **Uplink utilization:** Aggregate traffic on LAG members
* **Per-port errors:** CRC errors, input errors, output drops
* **Queue statistics:** Output queue drops, flow-control pause frames
* **Control plane:** CPU utilization, MAC table size, ARP table size

### Alerting thresholds:

* **Warning:** Sustained utilization **>50–60%** during business hours
* **Critical:** Sustained utilization **>70–80%** during business hours
* **Plan upgrade:** Any critical threshold breach or consistent warning alerts

### Validation workflow:

1. **Deploy telemetry:** Enable sFlow or NetFlow on uplinks and key stack members
2. **Baseline period:** Collect 30–90 days of data before making design changes or assumptions
3. **Analyze patterns:** Peak hour traffic, burst patterns, east-west vs north-south ratios
4. **Failure testing:** Simulate single-link and single-node failures; verify:
   - Traffic converges within acceptable time (typically <1 second for L2, <10 seconds for L3)
   - Remaining capacity ≥ minimum requirements
   - No unexpected control plane issues (MAC flapping, route flapping)

### Vendor-specific commands (examples):

**Aruba CX:**
```
show vsf
show vsf link
show interface brief
show vsx status
show vsx brief
show lacp interfaces
```

**Ruckus ICX:**
```
show stack
show stack detail
show interfaces brief
show lag
show inline power
show mac-address
```

---

## Design templates (copy/paste for quotes)

### Small IDF (≤200 × 1G ports)

* **Edge ports:** 200× 1G access (typically 4× 48-port switches)
* **Uplink:** **1×10G** per IDF to MDF (or 2×10G for redundancy)
* **Stack:** **40G aggregate** minimum (options: 1×40G backbone, 2×10G + 1×40G, or 4×10G ring)
* **MDF switch requirements:** Non-blocking fabric ≥20–40 Gbps; forwarding ≥30–60 Mpps
* **Typical use case:** Single-building floor, small branch office

### Medium IDF (200–400 × 1G ports)

* **Edge ports:** 400× 1G access (typically 8× 48-port switches, possibly 2 stacks of 4)
* **Uplink:** **2×10G LAG** per IDF or **1×40G** per IDF to MDF
* **Stack:** **≥80G aggregate** minimum (prefer 40G per-hop; options: 2×40G ring, 4×40G mesh)
* **MDF switch requirements:** Non-blocking fabric ≥120–320 Gbps depending on number of IDFs; forwarding ≥90–480 Mpps
* **Typical use case:** Multi-floor building, medium campus, regional office

### Large IDF (>400 × 1G ports)

* **Edge ports:** 400–800× 1G access
* **Uplink:** **1×40G or 2×25G** per IDF; consider multiple uplinks for very large stacks
* **Stack:** **≥160G aggregate** or split into multiple smaller stacks with MLAG/VSX aggregation
* **MDF switch requirements:** Consider chassis-based switches or high-density fixed switches; fabric ≥400 Gbps; forwarding ≥600 Mpps
* **Architecture:** Strongly recommend **VSX/MLAG** distribution pair; consider spine-leaf if >1000 ports
* **Typical use case:** Large campus core, data center edge, enterprise headquarters

---

## Quick checklist for each design review (one-liner checks)

* [ ] Count total 1G edge ports and expected growth (12–36 month horizon)
* [ ] Choose oversubscription ratio (20:1 default) and compute required uplink bandwidth
* [ ] Verify stacking links meet ≥recommended stack bandwidth; confirm per-hop speed is adequate
* [ ] Distribute uplinks across stack members; prefer dual-homing or MLAG/VSX where warranted
* [ ] Check MDF switch specifications: fabric capacity, forwarding rate (Mpps), TCAM size, MAC/ARP/route table sizes
* [ ] Validate PoE budget versus expected device load (APs, phones, cameras, IoT)
* [ ] Add monitoring (sFlow/NetFlow, SNMP) and configure alert thresholds (50-60% warning, 70-80% critical)
* [ ] Document failure test plan and maintenance/upgrade path (ISSU capability, failover testing)
* [ ] Verify physical infrastructure: rack space, power circuits, cooling, cable management
* [ ] Review security posture: port security, DHCP snooping, dynamic ARP inspection, control plane policing

---

## Final notes & recommended next steps

1. **Populate a reference database** (Google Sheets, Airtable, or lightweight DCIM tool) with your preferred SKUs and these key fields:
   - Model number and SKU
   - Port count (1G/10G/25G/40G/100G breakdown)
   - PoE budget (total watts, PoE+, PoE++ support)
   - Uplink types and quantities
   - Stacking link types and maximum aggregate bandwidth
   - Switching fabric capacity and forwarding rate (Mpps)
   - List price and typical discounted price
   - Template use case (small/medium/large IDF, MDF/core)

2. **Pilot test** a medium-sized site (200–400 ports) with full telemetry (sFlow/NetFlow) for 60–90 days. Validate:
   - Actual oversubscription ratios match assumptions (20:1 for office)
   - Stack link utilization remains <50% during peak hours
   - ISL/peer-link behavior under load
   - Failover convergence times and traffic impact

3. **For deployments >200 ports or many IDFs (>4)**, propose **VSX/MLAG** distribution pair to your customer or management. Benefits:
   - Reduces stack backbone pressure and single-point-of-failure risks
   - Improves L3 routing behavior and convergence
   - Enables hitless maintenance windows with ISSU
   - Provides better scalability path for future growth

4. **Automation readiness:** Begin planning for configuration management automation (Ansible, Python/Netmiko, vendor APIs) to ensure consistency across growing infrastructure and reduce manual configuration errors.

5. **Future-proofing:** Monitor industry trends toward 25G/50G access uplinks and 100G/400G core links. Plan optics and switch refresh cycles to align with these transitions over the next 3-5 years.

---

## Additional resources & references

* Aruba CX VSX Configuration Guide: [Aruba Support Portal](https://www.arubanetworks.com/techdocs/)
* Ruckus ICX Stacking and MCLAG Guide: [Ruckus Support](https://support.ruckuswireless.com/)
* IEEE 802.3 Ethernet Standards: [IEEE Standards Association](https://standards.ieee.org/)
* Network Design Best Practices: Cisco, Juniper, Aruba design guides (vendor-neutral principles apply)

---

*This document is a living reference—update it as you deploy and learn from real-world telemetry. Design conservatively, monitor actively, iterate intelligently.*