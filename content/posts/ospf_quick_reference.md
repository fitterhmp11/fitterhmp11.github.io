---
title: "OSPF Quick Reference — Key Principles & Best Practices"
date: 2026-02-28T01:00:00
tags:
  - ospf
  - lab
  - aruba-cx

categories: 
- networking

draft: false
url: "ospf-quick-reference"
summary: "OSPF Quick Reference — Key Principles & Best Practices"
description: "OSPF Quick Reference — Key Principles & Best Practices"
---



# OSPF Quick Reference — Key Principles & Best Practices

---

## Adjacency & Neighbor Formation

- **Adjacency does not mean correctness.** OSPF will reach FULL state even with mismatched network types. Verify your design — don't trust neighbor state alone.
- **The Dead timer governs neighbor loss detection, not link state.** If OSPF goes down on one side, the remote neighbor persists until its Dead timer expires (default 40s). Dead timers must match on both sides.
- **Point-to-point converges faster than broadcast.** Broadcast waits a full Dead interval (40s) for DR/BDR election. Use point-to-point on all direct router-to-router links.
- **Passive interfaces stop Hellos locally — immediately.** The local router drops the neighbor at once. The remote side waits for the Dead timer. Neighbor loss is detected independently on each side.

---

## Router IDs & Loopbacks

- **Always assign a router ID explicitly.** Without one, OSPF picks the highest IP — which can change. Instability here causes unnecessary LSDB churn.
- **Use /32 loopbacks for router IDs.** Loopbacks don't flap. They provide stable management endpoints (SSH, SNMP, ping) and reliable source addresses.
- **A changed router ID does not immediately clean up the LSDB.** The old LSA ages out at MaxAge (3600s). Plan router ID changes during maintenance windows.

---

## Cost & Path Selection

- **Cost is outbound only.** You control what your router advertises — not what your neighbor prefers. Applying cost to one side creates asymmetric routing.
- **To enforce symmetric routing: set cost on both ends of a link.**
- **Space costs far enough apart that no number of hops can overturn your intent.**

|Role|Recommended Cost|
|---|---|
|Primary transit|10|
|Secondary transit|100|
|Tertiary / backup|1000|
|Avoid unless necessary|5000|

- **Reference bandwidth on Aruba CX is 100,000 Mbps.** A 1G link = cost 100 by default. Adjust reference bandwidth to reflect your actual link speeds before deploying.
- **OSPF cost is a design tool — it does not self-optimize.** Desired path behavior requires explicit configuration.

---

## Route Selection Hierarchy

When multiple OSPF sources advertise the same destination, this order is absolute and non-negotiable:

1. **Intra-area (O)** — always wins
2. **Inter-area (O IA)**
3. **External Type 1 (O E1)**
4. **External Type 2 (O E2)**

> **Type beats cost. E1 always beats E2, regardless of absolute metric values.**

---

## DR/BDR & Network Types

- **DR/BDR exists to reduce adjacency count on multi-access segments.** Without it, a segment with _n_ routers requires _n(n−1)/2_ adjacencies. DR reduces this to _n_.
- **DROthers only reach Full state with the DR and BDR — never with each other.** DROther-to-DROther adjacency stays at 2-Way.
- **Use point-to-point on direct links. Use broadcast only when a segment genuinely has multiple routers.**
- **DR election is not preemptive by default.** The first router up wins. If you care about which router is DR, set priority explicitly — and do it before OSPF starts.

---

## LSDB Fundamentals

- **All routers in an area must maintain an identical LSDB.** Each router then runs SPF independently, placing itself at the center.
- **Routes exist because LSAs exist.** Before asking "why is this route missing," ask "why is the LSA missing."
- **Passive interfaces are not invisible.** They appear as stub networks in Type 1 LSAs, contribute to SPF, and are counted in summary metrics. Passive means no Hellos — not no advertisement.
- **Redistributing connected routes creates Type 5 LSAs, not stub links.** Prefer passive interfaces for directly connected networks — it keeps the routes intra-area and the LSDB clean.

---

## Multi-Area Design

- **Areas limit topology knowledge — not reachability.**
- **All non-backbone areas must connect to Area 0.** Without a path to Area 0, inter-area routes vanish. This is not configurable away — it is fundamental to how OSPF works.
- **An ABR has one brain but multiple LSDBs.** It learns routes in one area and originates Type 3 Summary LSAs into the others, always advertising itself as the originator.
- **The ABR is the trust boundary between areas.** Routers in other areas cannot see inside — they only see what the ABR tells them.

---

## Area Types — One-Line Rules

|Area Type|What Gets Blocked|Use Case|
|---|---|---|
|Normal|Nothing|Backbone and areas needing full visibility|
|Stub|Type 5 (external)|Branch with no local external connections|
|Totally Stubby|Type 5 + Type 3|Low-resource edge — one exit, no need to know routes|
|NSSA|Type 5 (allows Type 7)|Branch with a local external connection (partner, ISP)|
|Totally NSSA|Type 5 + Type 3 (allows Type 7)|Most efficient hybrid branch design|

**Rules that never change:**

- Type 1 & 2 are **never blocked** — routers always need them.
- Any area with "Stub" or "NSSA" **always blocks Type 5.**
- The "Totally" keyword **kills Type 3s**, leaving only a default route.
- Type 7 is **NSSA only**. The ABR translates it to Type 5 before it crosses into Area 0.

---

## Stub vs. NSSA — The Decision

- **Stub:** No external connections in that area. Simple. Use it.
- **NSSA:** The area needs to _inject_ external routes locally (partner link, static redistribution) while still behaving like a stub toward the rest of the domain.
- **Totally variants:** Add `no-summary` at the ABR to block Type 3s and reduce the routing table to just local routes + one default. Best for resource-constrained edge routers.

---

## Route Summarization

- **Summarization is a design decision, not a tuning knob.** Decide before you deploy. Changing it later is disruptive.
- **If an area has multiple ABRs, summarize consistently on all of them.** ABRs do not coordinate. Inconsistent summarization produces different routes in the backbone from each ABR, causing asymmetric paths and debugging pain.
- **Summaries mask failures.** A summary stays advertised as long as _any_ component subnet exists. The backbone will not know a specific subnet failed. This is expected behavior — plan for it with monitoring inside the area.
- **Summarization does not help the area being summarized.** It only reduces the backbone's LSDB. The summarized area still runs full SPF.
- **Summary metrics represent worst-case cost (Aruba CX).** The highest contributing route's cost becomes the summary cost — conservative by design.
- **Keep loopbacks outside summary ranges.** A high-cost loopback inside the range inflates the summary metric for every router in the backbone.

> **Summarization protects the backbone. It trades per-subnet visibility for scale and stability.**

---

## External Routes & Redistribution

- **E2 (default): flat cost.** The metric never increases as the route traverses hops. Use when there is one exit and internal distance doesn't matter.
- **E1: cumulative cost.** Internal path cost is added at every hop. Use when you have multiple exits and want routers to choose the closest one.
- **E1 always beats E2 for the same prefix — regardless of actual metric values.** This is not configurable.
- **Control redistribution with prefix-lists and route-maps.** Never redistribute broadly without filtering. Uncontrolled redistribution pollutes the LSDB and makes troubleshooting painful.
- **Use route tags to prevent redistribution loops.** In multi-domain or M&A scenarios: tag routes on entry, deny those tags at all other redistribution points.

---

## Default Route

- **`default-information originate`** injects a Type 5 LSA for `0.0.0.0/0`. The router must have a default route in its own table (unless `always` is specified).
- **Multiple ASBRs originating a default route:** routers choose based on SPF cost to the ASBR. Tune the metric or use E1 to influence which exit is preferred.
- **Stub and Totally Stubby areas generate a default route automatically (Type 3).** You do not need to configure it.
- **NSSA does not auto-generate a default route** unless explicitly configured or `no-summary` (Totally NSSA) is applied.

---

## Convergence & Stability

- **Convergence is a sequence:** link down → neighbor loss → LSA update → SPF → routing table change. Each step takes time.
- **BFD collapses failure detection from 40 seconds to milliseconds.** It runs independently of OSPF and notifies OSPF the moment a path fails.
    - Enable on both sides of a link.
    - Do **not** use on unstable or frequently flapping links — rapid BFD events trigger repeated SPF and destabilize the domain.
- **Stability matters more than convergence speed.** A link that flaps every few seconds causes more damage than a 40-second convergence time.
- **Graceful Restart keeps traffic flowing during a control-plane restart.** Use it on chassis platforms where the supervisor can restart independently of the forwarding hardware.

---

## Authentication

- **Always authenticate OSPF inter-router links in production.** Unauthenticated OSPF accepts any router as a neighbor.
- **MD5 is the practical choice for OSPFv2.** Plain text offers no real protection. SHA requires OSPFv3.
- **Authentication type and key must match on both sides.** A mismatch silently prevents adjacency formation.
- **Using the same key across all routers in an area is standard practice.**

---

## Troubleshooting Mental Model

| Symptom                     | First Question                                                          |
| --------------------------- | ----------------------------------------------------------------------- |
| Missing route               | Is the LSA present in the LSDB?                                         |
| Adjacency stuck in Exchange | Do MTUs match?                                                          |
| Neighbor not forming        | Do Hello/Dead timers and area IDs match?                                |
| Unexpected path             | What are the actual costs on both ends? Is routing symmetric?           |
| Route flapping              | Is a link flapping? Is BFD enabled on an unstable link?                 |
| Summary not appearing       | Is summarization configured on **all** ABRs?                            |
| External route missing      | Is the area type blocking Type 5? Is the ASBR redistributing correctly? |

> **"Routes exist because LSAs exist. Debug the LSDB, not the routing table."**

---

_Derived from Aruba CX OSPF home lab — Containerlab._