---
title: "OSPF Home Lab Reference"
date: 2026-02-28T02:00:00
tags:
  - ospf
  - lab
  - aruba-cx

categories: 
- networking

draft: false
url: "ospf-home-lab-reference"
summary: "OSPF Home Lab Reference"
description: "OSPF Home Lab Reference"
---
# OSPF Home Lab — Study Notes & Reference

**Platform:** Aruba CX (VMs) in Containerlab **Topology:** 3-router triangle — cx1 (core / ABR candidate), cx2 & cx3 (edge), Linux hosts on cx2 and cx3 **Purpose:** Personal knowledge base capturing lab observations, key insights, and OSPF reference material.

---

## Table of Contents

1. [Phase 0 – Foundations & Protocol Intuition](#phase-0--foundations--protocol-intuition)
    
    - [Lab 0.1 – OSPF Adjacency Fundamentals](#lab-01--ospf-adjacency-fundamentals)
        
    - [Lab 0.2 – Network Type Mismatch](#lab-02--network-type-mismatch)
        
    - [Lab 0.3 – Passive Interface Behavior](#lab-03--passive-interface-behavior)
        
2. [Phase 1 – Single-Area Routing Behavior](#phase-1--single-area-routing-behavior)
    
    - [Lab 1.1 – Intra-Area Route Learning](#lab-11--intra-area-route-learning)
        
    - [Lab 1.2 – Failure & Convergence](#lab-12--failure--convergence)
        
3. [Phase 2 – Parallel Links & Path Selection](#phase-2--parallel-links--path-selection)
    
    - [Lab 2.1 – Equal-Cost Multi-Path (ECMP)](#lab-21--equal-cost-multi-path-ecmp)
        
    - [Lab 2.2 – Unequal Cost Paths](#lab-22--unequal-cost-paths)
        
4. [Phase 3 – Network Types & Flooding Behavior](#phase-3--network-types--flooding-behavior)
    
    - [Lab 3.1 – Broadcast Networks & DR/BDR](#lab-31--broadcast-networks--drbdr)
        
    - [Lab 3.2 – Point-to-Point Comparison](#lab-32--point-to-point-comparison)
        
5. [Phase 4 – Multi-Area OSPF](#phase-4--multi-area-ospf)
    
    - [Lab 4.1 – Introducing a Second Area](#lab-41--introducing-a-second-area)
        
    - [Lab 4.2 – Backbone Dependency](#lab-42--backbone-dependency)
        
6. [Phase 5 – Stub & NSSA Areas](#phase-5--stub--nssa-areas)
    
    - [Lab 5.1 – Stub Area](#lab-51--stub-area)
        
    - [Lab 5.2 – NSSA](#lab-52--nssa)
        
7. [Phase 6 – Summarization & Design Trade-offs](#phase-6--summarization--design-trade-offs)
    
    - [Lab 6.1 – Route Summarization](#lab-61--route-summarization)
        
8. [Phase 7 – Failure, Scale & Design Thinking](#phase-7--failure-scale--design-thinking)
    
    - [Lab 7.1 – Failure Injection](#lab-71--failure-injection)
        
    - [Lab 7.2 – Design Reflection](#lab-72--design-reflection)
        
9. [General OSPF Reference](#general-ospf-reference)
    
    - [OSPF Packet Types](#ospf-packet-types)
        
    - [LSA Types](#lsa-types)
        
    - [Area Types & LSA Filtering](#area-types--lsa-filtering)
        
    - [External Routes: E1 vs. E2](#external-routes-e1-vs-e2)
        
    - [Route Selection Hierarchy](#route-selection-hierarchy)
        
    - [OSPF Cost & Reference Bandwidth](#ospf-cost--reference-bandwidth)
        
    - [Authentication](#authentication)
        
    - [BFD (Bidirectional Forwarding Detection)](#bfd-bidirectional-forwarding-detection)
        
    - [Graceful Restart](#graceful-restart)
        
    - [Default Route Origination](#default-route-origination)
        
    - [Redistribution & Route Maps](#redistribution--route-maps)
        
    - [ABR Summarization — Design Rules](#abr-summarization--design-rules)
        
    - [Real-World Design Scenarios](#real-world-design-scenarios)
        

---

## Phase 0 – Foundations & Protocol Intuition

### Lab 0.1 – OSPF Adjacency Fundamentals

**Objective:** Understand how OSPF forms adjacencies and what conditions are required.

**Lab Setup:**

- Enabled OSPF on inter-router links only
    
- Placed all routers in a single area (Area 0)
    
- Assigned stable router IDs via loopbacks
    

**Observations:**

- cx1 was configured as point-to-point, cx2 as broadcast — adjacency still came up and reached FULL.
    
- New neighbor took approximately 40 seconds to transition from 2-Way to FULL. This is because on a broadcast network, OSPF waits one full Dead interval (40s) before completing DR/BDR election. Point-to-point is significantly faster.
    
- Both sides must agree on the Dead timer interval for the adjacency to remain stable.
    
- On the remote side, the neighbor entry persists until the Dead timer expires — even if OSPF is disabled on the other router.
    

> _OSPF adjacency can reach FULL even when network types are mismatched. This is tolerated in simple topologies but introduces hidden failure modes when scaling or adding routers._

**Key Insight:** OSPF adjacency formation is symmetric only when conditions match.

---

#### Loopback Interfaces in OSPF

- Use `/32` prefix for loopback addresses.
    
- Multiple loopbacks can exist on the same router (e.g., for different VRFs).
    
- OSPF treats a loopback as a local (stub) route.
    
- Loopbacks must be added to OSPF-routed networks to be advertised.
    
- A loopback interface is always up unless administratively disabled — no flapping.
    
- If no router ID is manually assigned, OSPF selects the highest IP address on the router.
    
- Useful for management and monitoring (ping, SSH, SNMP) and as a stable source address for routing.
    

#### Router ID Change Behavior

When a router ID is changed, the old Router LSA is not immediately removed from the LSDB. OSPF tracks LSA age, and the key timer is **MaxAge** (3600 seconds). When an LSA reaches MaxAge without being refreshed, it is flushed from the LSDB. MaxAge is an OSPF standard value and is not configurable.

---

### Lab 0.2 – Network Type Mismatch

**Objective:** Learn that OSPF can appear healthy even when design assumptions differ.

**Lab Setup:**

- One side of a link configured as point-to-point, the other as broadcast
    

**Observations:**

- Adjacency still reached FULL despite the mismatch.
    
- Each router described the interface differently in its LSA.
    

**Key Insight:** OSPF prioritizes adjacency formation over design correctness. A mismatch here will not prevent the neighbor from forming but can cause subtle issues at scale (e.g., incorrect DR/BDR behavior, unexpected flooding).

---

### Lab 0.3 – Passive Interface Behavior

**Objective:** Understand what passive interfaces actually do.

**Lab Setup:**

- One inter-router interface set to passive
    

**Observations:**

- The router with the passive interface immediately removed the neighbor.
    
- The remote router kept the neighbor entry until the Dead timer expired.
    
- Neighbor loss is detected locally and independently on each router.
    

**Key Insight:** Passive interfaces stop Hello transmission, which means the local router drops the neighbor immediately. The remote side only discovers the loss when the Dead timer fires.

---

## Phase 1 – Single-Area Routing Behavior

### Lab 1.1 – Intra-Area Route Learning

**Objective:** Understand how routes appear in a single OSPF area.

**Lab Setup:**

- Advertised all inter-router links and host-facing subnets
    

**Observations:**

- All routes appeared as intra-area (`O`) routes.
    
- Metrics reflect link costs, not hop count.
    
- On Aruba CX, the `network` command is not used. OSPF is enabled per-interface by marking interfaces as active or passive, or by redistributing connected/static routes.
    
- Loopback and host VLANs are not automatically redistributed on cx3.
    

> _Connected networks should be advertised by enabling OSPF on the interface and marking it passive — not by redistribution._

**Passive Interface and LSDB Visibility:**

The passive interface subnet does appear in the LSDB, even though no Hellos are sent on that interface. Here is why:

1. Each router builds a Router LSA (Type 1) describing its router ID, active neighbor links, and stub networks (passive interfaces).
    
2. These LSAs are flooded to all other routers in the area across active inter-router links.
    
3. When cx1 and cx2 receive cx3's Router LSA, they store it in their LSDB and run SPF.
    
4. SPF sees cx3 as a node with links to the inter-router links and the stub networks (loopback, VLAN 3).
    
5. SPF computes shortest-path routes to each stub network and installs them in the routing table.
    

**Redistribution vs. Passive Interface:**

If you use `redistribute connected`, the LSDB changes. The connected subnets appear as AS External LSAs (Type 5), not as stub links in a Router LSA. **This is generally not preferred for directly connected networks.**

**Key Commands:**

- `show ip ospf` — Displays OSPF process info, area checksum, and SPF calculation count. The checksum sum for an area is identical across all routers in that area. SPF calculation count will vary per router.
    

**Key Insight:** All routers in an area maintain an identical LSDB. Each router then independently runs SPF, placing itself at the center of the topology.

---

### Lab 1.2 – Failure & Convergence

**Objective:** Observe how OSPF reacts to a link failure.

**Lab Setup:**

- Shut down one inter-router link
    

**Observations:**

- Convergence followed a clear sequence: neighbor loss → LSA update → SPF recalculation → routing table update.
    
- Traffic from Linux hosts was briefly interrupted, then resumed via the alternate path.
    

**Key Insight:** Convergence is a sequence of events, not a single atomic action. Understanding the order helps with diagnosing convergence issues.

---

## Phase 2 – Parallel Links & Path Selection

### Lab 2.1 – Equal-Cost Multi-Path (ECMP)

**Objective:** Understand how OSPF installs multiple paths.

**Lab Setup:**

- Created parallel links between routers with equal cost
    

**Observations:**

- Multiple next hops were installed in the routing table for the same destination.
    
- Shutting down one parallel link caused no ping interruption — OSPF switched to the remaining path very quickly.
    

**Key Insight:** ECMP improves resiliency more than raw throughput. In practice, OSPF per-flow load balancing distributes sessions, not individual packets.

---

### Lab 2.2 – Unequal Cost Paths

**Objective:** Learn how cost influences path preference.

**Lab Setup:**

- Set both interfaces connecting cx2 and cx3 to cost 1000, forcing all traffic through cx1
    

**Observations:**

- All traffic routed through cx1. The triangle effectively became a "V" from SPF's perspective.
    
- After failing the preferred path, OSPF activated the backup path.
    

**Additional Notes on Interface Cost:**

- Cost is strictly **outbound**. When you change the cost on an interface, **you affect only what the local router advertises in its Router LSA for that link.**
    
- To ensure symmetric routing, set cost on **both** interfaces of a link.
    
- Set cost on only one interface if you intentionally want asymmetric routing (advanced use case).
    

**Cost Influence Rules:**

- Influence traffic **leaving** Router A → change cost on Router A's interface
    
- Influence traffic **toward** Router A → change cost on all neighbors pointing to A
    
- Enforce primary/backup → change cost on both ends of both links
    
- Fine-tune ECMP → change cost on one side only
    

**Recommended Cost Spacing (Example):**

| ROLE                   | COST |
| ---------------------- | ---- |
| Primary transit        | 10   |
| Secondary transit      | 100  |
| Tertiary / backup      | 1000 |
| Avoid unless necessary | 5000 |

> _OSPF costs should be spaced far enough apart that no reasonable number of additional hops can overturn the intended path preference._

**Key Insight:** OSPF cost is a design tool, not an automatic optimization. It requires intentional configuration to produce desired behavior.

---

## Phase 3 – Network Types & Flooding Behavior

### Lab 3.1 – Broadcast Networks & DR/BDR

**Objective:** Understand why DR/BDR exists.

**Lab Setup:**

- Configured a shared segment as broadcast with at least three routers
    

**Observations:**

- DR and BDR roles were elected. Only the DR formed Full adjacencies with all other routers.
    
- DROther routers stay at 2-Way state with each other.
    
- LSAs are flooded via the DR.
    

**Captured Packet — Broadcast Hello:**

21:28:07.090753 IP 10.3.3.1 > ospf-all.mcast.net: OSPFv2, Hello, length 44  
    Router-ID 172.16.1.3, Backbone Area, Authentication Type: none (0)  
    Options [External]  
    Hello Timer 10s, Dead Timer 40s, Mask 255.255.255.0, Priority 1  
    Designated Router 10.3.3.1

**Multicast Addresses:**

- `224.0.0.6` — Used by DROthers to send updates to the DR/BDR
    
- `224.0.0.5` — Used by the DR to flood updates to all OSPF routers
    

**Key Insight:** DR/BDR reduces the number of adjacencies from O(n²) to O(n), optimizing control-plane scalability on multi-access segments.

---

### Lab 3.2 – Point-to-Point Comparison

**Objective:** Contrast broadcast and point-to-point behavior.

**Lab Setup:**

- Converted the same segment to point-to-point
    

**Observations:**

- No DR/BDR election occurred.
    
- Simpler adjacency model — both routers go directly to Full.
    
- LSDB was simpler; no Type 2 Network LSA generated.
    

**Captured Packet — Point-to-Point Hello:**

21:29:00.598392 IP 10.3.3.1 > ospf-all.mcast.net: OSPFv2, Hello, length 44  
    Router-ID 172.16.1.3, Backbone Area, Authentication Type: none (0)  
    Options [External]  
    Hello Timer 10s, Dead Timer 40s, Mask 255.255.255.0, Priority 1  
    (No Designated Router field)

**Captured Packet — Point-to-Point with MD5 Authentication:**

21:30:40.607241 IP 10.3.3.1 > ospf-all.mcast.net: OSPFv2, Hello, length 44  
    Router-ID 172.16.1.3, Backbone Area, Authentication Type: MD5 (2)  
    Key-ID: 1, Auth-Length: 16, Crypto Sequence Number: 0x0000119d  
    Options [External]  
    Hello Timer 10s, Dead Timer 40s, Mask 255.255.255.0, Priority 1

**Key Insight:** Point-to-point eliminates DR/BDR complexity and is the preferred network type for direct router-to-router links. Use broadcast only when a segment genuinely has multiple routers.

---

## Phase 4 – Multi-Area OSPF

### Lab 4.1 – Introducing a Second Area

**Objective:** Understand LSDB separation and ABR behavior.

**Lab Setup:**

- cx2 placed in Area 0
    
- cx3 placed in Area 10
    
- cx1 acts as ABR (participating in both Area 0 and Area 10)
    

**Observations:**

- When cx3 was placed in Area 10, a new LSA type appeared in the LSDB: the **Inter-Area Summary LSA (Type 3)**.
    
- The Type 3 LSA lists the subnets of the other area, with the ABR's router ID as the Advertising Router.
    
- cx1 appeared as the Advertising Router for all Type 3 Summary LSAs in both Area 0 and Area 10.
    
- Route types in the routing table changed: `O` (intra-area) for same-area routes, `O IA` (inter-area) for cross-area routes.
    

> _cx1 is an ABR because it participates in Area 0 and Area 10. It originates Summary (Type 3) LSAs into each area describing networks from the other area, and is the Advertising Router for those LSAs._

> **An ABR has one brain but multiple LSDBs. It learns routes in one area and advertises summaries of them into the other areas, always listing itself as the Advertising Router.**

**Key Insight:** Areas limit topology knowledge, not reachability. Routers in different areas can still reach each other via Type 3 LSAs from the ABR.

---

### Lab 4.2 – Backbone Dependency

**Objective:** Internalize why Area 0 is mandatory.

**Lab Setup:**

- Broke Area 10's connection to Area 0
    

**Observations:**

- Inter-area routes disappeared from the routing table.
    
- Intra-area routes within each area remained intact.
    

**Key Insight:** Area 0 (the backbone) is the transit point for all inter-area routing. Non-backbone areas must connect to Area 0, either directly or via a virtual link.

---

## Phase 5 – Stub & NSSA Areas

### Lab 5.1 – Stub Area

**Objective:** Reduce LSDB complexity at the edge.

**Lab Setup:**

- Converted Area 10 to a stub area
    

**Observations:**

- The only change visible on cx3 was the addition of one extra Summary LSA: a `0.0.0.0/0` default route injected by the ABR.
    
- External LSAs (Type 5) were no longer present in Area 10's LSDB.
    
- All internal (Type 1) and inter-area (Type 3) routes remained unchanged.
    

> _Converting an area to stub replaces all external routes with a single default route (0.0.0.0/0) injected by the ABR. All internal and inter-area routes remain unchanged. This reduces LSDB size and speeds SPF calculations inside the stub area._

**Key Insight:** Less information can improve stability. A stub area trades external route visibility for a smaller, more stable LSDB.

---

### Lab 5.2 – NSSA

**Objective:** Allow controlled external route injection from within a stub-like area.

**Lab Setup:**

- Converted Area 10 to NSSA
    
- Added a static route on cx3 and redistributed it into OSPF
    

**Observations:**

- When Area 10 was converted to NSSA, the Type 3 default route (injected by the ABR in stub mode) disappeared and was replaced by a **Type 7 NSSA External LSA** for the default route — even before any redistribution was configured. This is a placeholder Type 7 default, automatically generated so the ABR can translate it to a Type 5 for the backbone if needed.
    
- After redistributing the static route on cx3:
    
    - cx3 LSDB: Type 7 NSSA External LSA for `10.50.50.x`, ADV router = `172.16.1.3` (cx3)
        
    - cx2 LSDB: Type 7 NSSA External LSA for `10.50.50.x` (ADV = cx3) **and** Type 5 External LSA for `10.50.50.x` (ADV = cx2)
        
    - cx1 LSDB: Type 5 External LSA for `10.50.50.x`, ADV router = `172.16.1.2` (cx2, the ABR)
        
- cx2 serves as the ABR for Area 10 and translates the Type 7 LSA from the NSSA into a Type 5 LSA for distribution to Area 0 and the rest of the OSPF domain.
    

**Key Insight:** NSSA exists for real edge-network use cases where a site needs both the LSDB reduction of a stub area and the ability to inject local external routes.

---

## Phase 6 – Summarization & Design Trade-offs

### Lab 6.1 – Route Summarization

**Objective:** Control routing table growth using ABR summarization.

**Lab Setup:**

- Created additional subnets in Area 10 on cx3: loopback2 (`10.99.99.1/32`) and VLAN 99 (`10.99.1.0/24`)
    
- Summarized these at the ABR (cx3) using:
    
    router ospf 1  
        area 0.0.0.10 range 10.99.0.0/16 type inter-area
    
- Later moved cx2 into Area 10, creating two ABRs
    

**Observations:**

- Before summarization: cx2's LSDB showed individual Type 3 LSAs for each subnet in Area 10.
    
- After summarization: cx2's LSDB showed only `10.99.0.0/16` as a single Type 3 LSA.
    
- With two ABRs (cx2 and cx3), and only cx3 summarizing: Area 0 received both the summary from cx3 and the specific routes from cx2 — inconsistent LSDB.
    
- After adding summarization to cx2 as well: Area 0 saw only the summary from both ABRs.
    
- After failing the cx1-cx3 link: summarized routes updated to use cx2 automatically.
    

**Route Types in `show ip ospf route`:**

- Area 0 routers: see both `Intra-area` and `Inter-area` routes
    
- Area 10 routers: see only `Intra-area` routes (because all routes are local to that area in this topology)
    

> _On Aruba CX, OSPF summary LSAs use the highest contributing intra-area cost when computing the summary metric. This ensures the advertised metric safely represents the worst-case path inside the summary range and avoids blackholing in multi-ABR designs._

> **If multiple ABRs serve an area, summarization must be configured consistently on all ABRs.**

**Key Insight:** Summarization is a design decision — it trades routing table precision for scalability and backbone stability. Summarization does not reduce SPF load inside the summarized area; it only benefits the backbone.

---

## Phase 7 – Failure, Scale & Design Thinking

### Lab 7.1 – Failure Injection

**Objective:** Stress OSPF stability under rapid link changes.

**Lab Setup:**

- Rapidly flapped links
    

**Observations:**

- Neighbor stability suffered during rapid flapping.
    
- Route churn was visible in the routing table.
    
- User-visible impact (packet loss, latency spikes) preceded full protocol recovery.
    

**Key Insight:** Stability matters more than raw convergence speed. An unstable link that constantly triggers SPF recalculation is worse than a slightly slower but stable failover.

---

### Lab 7.2 – Design Reflection

**Objective:** Transition from operator to designer.

**Reflection Points:**

- What breaks first as scale increases? — The LSDB grows, SPF runs more frequently, and flooding overhead increases.
    
- Where would you add hierarchy? — Additional areas with summarization at ABRs reduce backbone churn.
    
- When would you stop using OSPF? — When route scale exceeds what a single AS can handle efficiently, or when policy-based routing across administrative domains is required (consider BGP).
    

---

## General OSPF Reference

### OSPF Packet Types

OSPF uses five packet types to discover neighbors and synchronize the LSDB. None of these use TCP — reliability is handled by OSPF's own acknowledgment mechanism.

|TYPE|NAME|PRIMARY FUNCTION|
|---|---|---|
|1|Hello|Neighbor discovery, keepalives, DR/BDR election|
|2|DBD (Database Description)|"Table of contents" of the LSDB; used during initial sync|
|3|LSR (Link State Request)|Requests specific LSAs missing after DBD comparison|
|4|LSU (Link State Update)|Carries one or more LSAs; the actual data payload|
|5|LSAck|Confirms reliable receipt of an LSU|

**Adjacency Formation Sequence:**

1. **Down → Init:** Routers send Hellos to discover each other.
    
2. **ExStart → Exchange:** Routers exchange DBDs to compare their LSDB "table of contents."
    
3. **Loading:** Routers send LSRs for missing LSAs; neighbors respond with LSUs; LSAcks confirm receipt.
    
4. **Full:** Databases are identical. Full adjacency is established.
    

**MTU Sensitivity:** MTU must match on both sides. If a DBD or LSU exceeds the interface MTU, the packet is dropped, leaving the adjacency stuck in Exchange state.

---

### LSA Types

#### Internal Area LSAs (stay within the originating area)

- **Type 1 — Router LSA:** Generated by every OSPF router. Describes the router's ID, its active links to neighbors, and its stub networks (passive interfaces/connected subnets).
    
- **Type 2 — Network LSA:** Generated by the DR on broadcast segments. Lists all routers attached to that segment.
    

#### Inter-Area and External LSAs

- **Type 3 — Summary LSA:** Generated by ABRs. Advertises subnets from one area into another. In totally stubby areas, filtered down to a single default route.
    
- **Type 4 — ASBR Summary LSA:** Generated by ABRs. Tells the rest of the network how to reach the ASBR (the router redistributing external routes).
    
- **Type 5 — AS External LSA:** Generated by the ASBR. Advertises routes redistributed from outside OSPF. Flooded everywhere except Stub and NSSA areas.
    
- **Type 7 — NSSA External LSA:** Used only inside an NSSA. Carries external routes originated within the NSSA. The ABR translates Type 7 to Type 5 when forwarding into Area 0.
    

#### LSA Maintenance

- **Sequence Numbers:** Start at `0x80000001`, incrementing by 1 for each topology change or the 30-minute periodic refresh.
    
- **LSA Checksums:** Each LSA carries a checksum. Routers compare individual LSA checksums (not a single global DB checksum) to detect discrepancies.
    
- **30-Minute Refresh:** Routers re-originate their own LSAs every 1800 seconds to keep the database healthy.
    
- **MaxAge:** If an LSA is not refreshed within 3600 seconds, it is purged from the LSDB. This value is an OSPF standard and is not configurable.
    

#### Convergence Triggers

|EVENT|DETECTION METHOD|RESULTING ACTION|
|---|---|---|
|Link failure|Physical/data-link down signal|Immediate LSU with updated Type 1 LSA (sequence +1)|
|Router power loss|Dead Interval expiry (40s default)|Neighbors timeout and update their LSAs to remove the link|
|LSA corruption|Checksum mismatch|Router discards LSA and sends an LSR for a fresh copy|

---

### Area Types & LSA Filtering

|AREA TYPE|TYPE 1 & 2|TYPE 3 (INTER-AREA)|TYPE 5 (EXTERNAL)|TYPE 7 (NSSA)|
|---|---|---|---|---|
|Normal / Area 0|Allowed|Allowed|Allowed|Blocked|
|Stub|Allowed|Allowed|Blocked|Blocked|
|Totally Stubby|Allowed|Blocked*|Blocked|Blocked|
|NSSA|Allowed|Allowed|Blocked|Allowed|
|Totally NSSA|Allowed|Blocked*|Blocked|Allowed|

*Totally Stubby and Totally NSSA still allow a single Type 3 default route (0.0.0.0/0) from the ABR.

**Area Type Descriptions:**

- **Normal (Standard):** Default area type. Supports all LSA types. Used for Area 0 and any area requiring full visibility.
    
- **Stub:** Blocks Type 5. ABR injects a Type 3 default route. Use for branch offices where external routes do not need to be known locally.
    
- **Totally Stubby:** Blocks Type 5 and Type 3 (except the default). Leaves only local routes and one exit. Use for low-resource edge routers.
    
- **NSSA:** Blocks Type 5 but allows Type 7 for local external route injection. Use when a site needs stub-like efficiency but also connects to an external network (e.g., a partner link).
    
- **Totally NSSA:** Blocks Type 5 and Type 3 (except default), allows Type 7. The most efficient design for a hybrid branch site.
    

**`no-summary` Command:** Applied at the ABR, this stops Type 3 Summary LSAs from being sent into an area, replacing them with a single default route. This converts a Stub to Totally Stubby, or an NSSA to Totally NSSA. Useful for remote sites with limited hardware where minimizing the routing table is a priority.

**Default Route Injection by Area Type:**

|AREA TYPE|DEFAULT ROUTE AUTO-GENERATED?|LSA TYPE|
|---|---|---|
|Normal|No — requires `default-information originate`|Type 5|
|Stub|Yes|Type 3|
|Totally Stubby|Yes|Type 3|
|NSSA|No — requires `area X nssa default-information-originate`|Type 7|
|Totally NSSA|Yes|Type 3|

**Key takeaways:**

- Type 1 & 2 are never blocked — routers always need them to map their immediate topology.
    
- Type 5 is always blocked in any area with "Stub" or "NSSA" in the name.
    
- Type 7 is NSSA-only. It is the only way to get external routes into a stubby environment.
    
- The "Totally" keyword blocks Type 3s, leaving only the default route.
    

---

### External Routes: E1 vs. E2

When redistributing external routes into OSPF, each route carries a metric type that determines how the cost is calculated as the route travels through the domain.

**External Type 2 (E2) — Default:**

- Total cost = external metric only. Internal path cost is ignored.
    
- The metric does not increase as the route traverses additional links.
    
- Use when there is a single exit point to the outside and internal distance to the ASBR is irrelevant.
    

**External Type 1 (E1):**

- Total cost = external metric + internal path cost to the ASBR.
    
- The metric increases with each hop toward the route's source.
    
- Use when there are multiple exit points and you want routers to choose the closest exit.
    
- E1 is always preferred over E2 for the same destination, regardless of absolute cost values.
    

**Selection Priority (hard-coded):**

1. Intra-area (`O`) — always wins
    
2. Inter-area (`O IA`)
    
3. External Type 1 (`O E1`)
    
4. External Type 2 (`O E2`)
    

On Aruba CX: `redistribute static metric-type type1`

> **Type First, Cost Second. E1 > E2 always. O > O IA always.**

---

### Route Selection Hierarchy

When a router learns the same subnet from multiple OSPF sources:

1. **Intra-Area (O)** — same area always wins
    
2. **Inter-Area (O IA)** — beats any external route
    
3. **External Type 1 (O E1)** — beats E2 every time
    
4. **External Type 2 (O E2)** — last resort for OSPF
    

---

### OSPF Cost & Reference Bandwidth

On Aruba CX, the reference bandwidth is configured as:

Reference Bandwidth: 100000 Mbps

This means a 1 Gbps link has a default cost of **100** (100,000 / 1,000 = 100).

Cost is strictly **outbound**. Changing the cost on an interface changes only what that router advertises in its Router LSA for that link — it does not affect how the remote router values the reverse path. To change the path for both directions of a conversation, cost changes must be applied at both ends of the link. Applying cost to only one side creates **asymmetric routing**.

---

### Authentication

Authentication between inter-router links is recommended in production environments. Typical practice is to use the same key for all routers in an area.

- OSPFv2 supports **plain text** and **MD5 message digest** authentication.
    
- SHA authentication requires **OSPFv3**.
    
- Authentication type and key must match on both sides of a link.
    

---

### BFD (Bidirectional Forwarding Detection)

BFD is a protocol that runs independently of OSPF and detects link or path failures in milliseconds — far faster than OSPF's default 10-second Hello / 40-second Dead timer cycle.

- When BFD detects a failure, it immediately tears down the OSPF adjacency and triggers SPF recalculation.
    
- Use BFD when sub-second failover is required.
    
- Do **not** use BFD on links that flap frequently — rapid BFD events will trigger repeated SPF recalculations and destabilize the network.
    
- BFD must be enabled on **both sides** of a link to form a session.
    

**On Aruba CX:**

1. Enable BFD globally
    
2. Enable BFD on each interface where it is needed
    

---

### Graceful Restart

Graceful Restart allows a router to restart its control plane (routing software) while the hardware data plane continues forwarding traffic. The restarting router signals its neighbors: _"I am restarting — please maintain the adjacency and keep forwarding during the restart window."_

Primarily used on chassis-based routers where the control plane (supervisor module) can restart independently of the line cards forwarding traffic.

**Observed on all three lab routers:**

- GR Status: enabled
    
- State: inactive
    
- Helper: enabled
    

---

### Default Route Origination

To inject a default route into the OSPF domain from a router that has a default route in its local routing table:

default-information originate

This generates a Type 5 LSA for `0.0.0.0/0` and advertises it to all normal areas.

**Default behavior:** Injects as External Type 2 (E2) with a cost of 1.

**Lab observation:** With both cx1 and cx2 configured to originate a default route, cx3 chose cx1's default because the forwarding cost to cx1's ASBR was lower (shorter SPF path). The metric of the originating router's `default-information originate` command can be tuned to influence which ASBR is preferred.

**Changing metric type for redistribution:**

A route-map can be used to set specific metric type and cost on redistributed routes. For example, changing a redistributed static route on cx1 to External Type 1 (E1) caused cx3 to always prefer cx1's version of that route over cx2's E2 version — regardless of absolute cost — because E1 always beats E2.

---

### Redistribution & Route Maps

To control which static routes are redistributed into OSPF, use a route-map with a prefix-list:

1. Create a prefix-list permitting only the desired routes.
    
2. Create a route-map that matches those prefixes with a `permit` statement.
    
3. The route-map's implicit `deny` at the end drops anything not matched.
    
4. Apply: `redistribute static route-map <name>`
    

**Metric Summary:**

|SETTING|DESCRIPTION|
|---|---|
|Default metric (e.g., 25)|Global starting cost for all external routes|
|Metric Type 2 (E2)|Flat cost — internal link costs are not added|
|Metric Type 1 (E1)|True cost — adds internal path cost to the seed metric|
|Route-map|Per-subnet control of cost, type, and tags|

**Route Tagging:**

Tags are metadata attached to redistributed routes. They are useful for loop prevention in multi-domain redistribution scenarios (e.g., M&A topologies). A router can be configured to deny redistribution of any route carrying a specific tag, preventing routes from "re-entering" the network from a second interconnect point.

---

### ABR Summarization — Design Rules

Summarization is configured at the ABR and affects only Type 3 (Inter-Area) LSAs. It does not affect Type 1, Type 2, Type 5, or Type 7 LSAs.

**Core rules:**

1. **Summarization is local and independent.** Each ABR summarizes based on its own view of the area. ABRs do not coordinate metrics or ranges.
    
2. **If all ABRs summarize, the backbone loses visibility.** Area 0 will never see specific routes and will not know which subnet failed. Summaries remain advertised as long as any component route exists.
    
3. **Summary metrics are conservative.** Aruba CX uses the highest contributing intra-area cost. The summary cost represents worst-case reachability, not best-path cost.
    
4. **Passive interfaces still contribute to summaries.** Passive does not mean invisible — passive interfaces still appear in Type 1 LSAs and affect summary metric selection.
    
5. **Loopbacks can skew summary metrics.** If a loopback with a high cost falls inside a summary range, it can inflate the advertised summary metric. Keep loopbacks outside summary ranges when possible.
    
6. **Inconsistent summarization causes routing anomalies.** Common mistakes: only one ABR summarizes; different ABRs use different ranges; overlapping summaries. Results include LSDB bloat, route flapping, suboptimal paths, and difficult debugging.
    
7. **Summarization does not reduce SPF inside the area.** The area being summarized still runs full SPF with the full LSDB. Only the backbone benefits from the reduced LSA count.
    
8. **Summaries can cause silent blackholes.** If one subnet inside a summary fails, the summary route remains advertised and traffic silently drops. Mitigation: careful address planning, smaller summaries, redundant ABRs, and monitoring inside areas.
    

> **ABR summarization is about protecting the backbone, not optimizing paths. It trades visibility for stability, and correctness for simplicity.**

**When to use summarization:**

- You want scale, failure isolation, and a cleaner backbone.
    
- You accept loss of per-subnet visibility and coarser metrics.
    

**When not to use summarization:**

- For traffic engineering purposes.
    
- To compensate for poor cost design.
    
- Without address planning aligned to summary boundaries.
    

---

### Real-World Design Scenarios

#### 1. Enterprise Dual-Homed Branch (ECMP)

**Topology:** Branch office connected to HQ via two equal-cost WAN links from different providers. **Technique:** ECMP — configure identical costs on both links. Both paths are active simultaneously. **Benefit:** Doubles effective throughput to HQ and provides automatic failover without complex configuration.

---

#### 2. Critical Financial Service Provider (BFD + Graceful Restart)

**Topology:** Core network where milliseconds of downtime impact thousands of transactions. **Technique:** BFD for sub-second failure detection; Graceful Restart to survive control-plane software updates. **Benefit:** BFD ensures failover in milliseconds when a fiber is cut. Graceful Restart ensures traffic continues forwarding during a planned control-plane restart, eliminating the need for a maintenance window.

---

#### 3. Multi-Exit Internet Gateway (E1 + Default-Information Originate)

**Topology:** Two data centers, each with its own ISP connection. **Technique:** External Type 1 (E1) with `default-information originate`. **Benefit:** Routers calculate the total cost to both exits. Traffic leaves through the geographically closest ISP exit, reducing latency and avoiding internal "tromboning."

---

#### 4. Merged Network — M&A Scenario (Route Maps + Prefix Lists + Tags)

**Topology:** Company A acquires Company B. Specific server subnets need to be shared between their OSPF domains. **Technique:** Redistribute only specific subnets using prefix-lists. Tag all redistributed routes with a unique value (e.g., `tag 888`). Configure all other interconnect routers to deny any incoming redistribution carrying that tag. **Benefit:** Only required subnets are shared. Tagged routes prevent routing loops at secondary interconnect points — a route cannot re-enter the network it originated from.

---

#### 5. Cloud-First Hybrid Network (Manual Cost + Floating Static)

**Topology:** On-premise network connected to AWS or Azure via VPN/Direct Connect. **Technique:** Set a high OSPF cost on the cloud path to keep traffic on the local fiber. Use a floating static route (higher administrative distance) as a fallback if the OSPF process itself fails. **Benefit:** Cloud egress is used only when the local path is unavailable. The floating static ensures reachability even if OSPF itself is down.

---

_Document generated from Aruba CX Containerlab OSPF home lab sessions._