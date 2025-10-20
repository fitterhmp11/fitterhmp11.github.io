---
title: "IGMP Overview with Ruckus ICX and VOIP Pageing"
date: 2025-10-20T01:00:00
tags:
  - concepts, igmp, ruckus, voip
categories: 
 - networking
draft: false
url: "igmp-ruckus"
summary: "GMP Overview with Ruckus ICX and VOIP Pageing"
description: "GMP Overview with Ruckus ICX and VOIP Pageing"
---

# Ruckus ICX Multicast and IGMP – Complete Reference

This document summarizes key concepts, terminology, best practices, and FAQs related to multicast, IGMP snooping, and VoIP paging on Ruckus ICX switches. Targeted for K-12 networks and small to medium enterprises.

**Note:** Command syntax is based on Ruckus FastIron 08.0.90+ firmware. Always verify commands against your specific firmware version.

---

## 1. Key Terms and Concepts

### VLAN

- A **Virtual LAN** separates traffic into broadcast domains.
- Multicast traffic within a VLAN can be controlled via IGMP snooping.

### Multicast

- A method to send **one copy of data to multiple receivers**.
- Destination IP addresses range from **224.0.0.0 to 239.255.255.255**.
- Useful for VoIP paging, IPTV, video conferencing, and mDNS discovery.

### Multicast Address Space

- **224.0.0.0 – 224.0.0.255**: Reserved for local network control (IGMP, routing protocols). **Not snooped by switches** - these are flooded to maintain control plane functionality.
- **224.0.1.0 – 238.255.255.255**: Globally-scoped and internetwork control applications. Includes well-known addresses (e.g., NTP at 224.0.1.1).
- **239.0.0.0 – 239.255.255.255**: Administratively-scoped (private) addresses for enterprise use like paging systems and internal IPTV.

**Important:** Addresses in the 224.0.0.x range are flooded by design and cannot be controlled via IGMP snooping.

### IGMP (Internet Group Management Protocol)

- A **Layer 3 protocol** used by hosts to join or leave multicast groups.
- Versions: IGMPv1, v2, v3 (v2 is most common for VoIP phones).
- Works **between hosts and queriers/routers** within a VLAN.
- IGMP messages are **control traffic**, not the multicast data itself.

### IGMP Snooping

- A **Layer 2 switch feature** that listens to IGMP control packets.
- Builds a **forwarding table mapping multicast groups → switch ports**.
- Forwards multicast traffic only to ports with interested receivers.
- **Requires an active querier** to maintain and refresh membership tables.
- Without snooping, multicast floods to all ports in the VLAN (inefficient but functional).

### IGMP Querier

- The device that **periodically sends IGMP Query messages** (typically every 125 seconds).
- Ensures switches maintain up-to-date multicast membership tables.
- Can be a Layer 3 router (your gateway) or a Layer 2 switch with an IP interface on the VLAN.
- Only **one active querier per VLAN**; if multiple queriers exist, the one with the **lowest IP address** wins the election.

### Active vs Passive Mode

- **Active Mode**: Switch acts as IGMP querier and sends query messages.
    - Use when: No router is present, or in standalone Layer 2 networks.
    - Only enable on **ONE switch** per VLAN.
- **Passive Mode**: Switch forwards queries but does not send them.
    - Use when: A router or another switch is the querier.
    - Default mode for most deployments.

### Multicast Router Port

- A port connected to a multicast router (PIM-enabled) that can **route multicast between VLANs/subnets**.
- **Only required** if multicast traffic must traverse multiple VLANs.
- **Not required** for single-VLAN multicast (e.g., VoIP paging within one voice VLAN).
- Can be learned automatically or configured statically.

### Static Group

- Manually configured multicast group on a switch.
- Forces multicast traffic for that group to specified ports.
- Useful for devices that **don't send IGMP join messages** (some legacy equipment).
- Acts as a proxy - the switch sends membership reports on behalf of the group.

### Fast Leave

- **fast-leave-v2** for IGMPv2: Switch immediately removes a port from the group when receiving an IGMP Leave message.
- No group-specific query is sent before removing the port.
- **Safe ONLY when**: One device per access port.
- **NEVER enable** on trunk ports or ports with multiple devices (e.g., IP phone + PC pass-through).

### Querier Address

- The IP address the switch uses to send IGMP queries when in active mode.
- Must be a **local IP address on that VLAN** (the VE interface IP).
- Configured automatically if you have a VE interface; manual configuration rarely needed.

---

## 2. How Multicast Works on a Layer 2 Switch

### Normal Operation Flow

1. **Querier sends IGMP General Queries** periodically to `224.0.0.1` (all-hosts multicast group).
2. Interested hosts respond with **IGMP Membership Reports** for the groups they want to join.
3. Switches with **IGMP snooping enabled**:
    - Listen to these control messages
    - Build a forwarding table: `multicast group → interested ports`
    - Forward multicast data traffic only to ports with active memberships
4. When membership queries stop or timeout:
    - Entries age out (typically 260 seconds)
    - Traffic stops being forwarded to those ports

### What Happens Without a Querier?

If IGMP snooping is **enabled** but there is **no active querier**:

1. Membership tables **age out** after ~260 seconds (2x query interval + grace period)
2. Once tables are empty, multicast behavior depends on switch configuration:
    - **Default behavior**: Multicast traffic is **dropped** (no forwarding)
    - **If flood-unregistered is enabled**: Multicast floods to all ports
3. **This is why a querier is critical** - without periodic queries, memberships cannot be maintained

### Example: VoIP Paging Flow

**Scenario:** VLAN 101, Voice VLAN, ICX Switch with VE interface as querier, IP phones on access ports

1. Switch (querier) sends **General Query** → `224.0.0.1` every 125 seconds
2. Phones respond with **Membership Reports** for paging group `239.10.10.1`
3. Switch updates forwarding table: `239.10.10.1 → ports 1-24`
4. Paging server sends audio stream to `239.10.10.1`
5. Switch forwards stream **only** to ports 1-24 (phones), not to uplinks or other ports
6. Querier continues sending periodic queries to maintain memberships

---

## 3. Ruckus ICX Configuration Best Practices

### Basic VLAN Multicast Configuration

Enable multicast on the VLAN (enables IGMP snooping in passive mode by default):

```bash
ICX7150(config)# vlan 101
ICX7150(config-vlan-101)# router-interface ve 101
ICX7150(config-vlan-101)# multicast passive
```

**Note:** `multicast passive` is the default when you enable multicast. Explicitly stating it is optional.

### Configuring Active Querier

To make a switch act as the IGMP querier (active mode):

```bash
ICX7150(config)# vlan 101
ICX7150(config-vlan-101)# multicast active
```

**Prerequisites:**

- Switch must have a **VE (Virtual Ethernet) interface** configured on the VLAN
- VE interface must have an IP address assigned

```bash
ICX7150(config)# vlan 101
ICX7150(config-vlan-101)# router-interface ve 101
ICX7150(config-vlan-101)# exit
ICX7150(config)# interface ve 101
ICX7150(config-if-ve-101)# ip address 10.10.101.1 255.255.255.0
```

The switch will use the VE interface IP (10.10.101.1) as the querier source address.

### Setting IGMP Version

Set IGMP version globally (applies to all VLANs unless overridden):

```bash
ICX7150(config)# ip multicast version 2
```

Or set per-VLAN:

```bash
ICX7150(config)# vlan 101
ICX7150(config-vlan-101)# multicast version 2
```

**Recommendations:**

- Use **IGMPv2** for most VoIP deployments (broadest compatibility)
- Use **IGMPv3** if you need source-specific multicast (SSM) or source filtering

### Enabling Fast Leave

Enable fast-leave for IGMPv2 on a VLAN:

```bash
ICX7150(config)# vlan 101
ICX7150(config-vlan-101)# multicast fast-leave-v2
```

**CRITICAL WARNING:**

- Only enable on **access ports with a single device**
- **Never enable** on trunk ports
- **Never enable** on ports with IP phones that have PC pass-through
- Fast-leave bypasses leave-wait timers - if multiple devices are on a port, traffic stops prematurely

### Configuring Static Groups

If you have devices that don't send IGMP joins, configure static groups:

```bash
ICX7150(config)# vlan 101
ICX7150(config-vlan-101)# multicast static-group 239.1.1.1 ethernet 1/1/1 to 1/1/10
```

This forces traffic for `239.1.1.1` to ports 1/1/1 through 1/1/10.

**Optional:** Disable static group proxy behavior (prevents switch from sending membership reports):

```bash
ICX7150(config)# vlan 101
ICX7150(config-vlan-101)# multicast static-grp-fwd-disable
```

### Configuring Static Router Ports

Force multicast traffic to specific uplink ports (useful for ensuring traffic reaches routers):

```bash
ICX7150(config)# vlan 101
ICX7150(config-vlan-101)# multicast router-port ethernet 1/3/1
```

### Global IGMP Snooping Enable/Disable

Enable IGMP snooping globally (usually enabled by default):

```bash
ICX7150(config)# ip multicast
```

Disable IGMP snooping globally (causes flooding - not recommended):

```bash
ICX7150(config)# no ip multicast
```

### Adjusting IGMP Timers

Modify query interval (default 125 seconds):

```bash
ICX7150(config)# ip multicast query-interval 60
```

Modify group membership age-out time (default 260 seconds):

```bash
ICX7150(config)# ip multicast group-membership-time 200
```

Modify leave-wait time for IGMPv2 (default 2 seconds):

```bash
ICX7150(config)# ip multicast leave-wait-time 1
```

---

## 4. Verification Commands

### Display Global IGMP Configuration

```bash
ICX7150# show ip multicast
```

**Sample Output:**

```
Summary of all vlans. Please use "sh ip mu vlan <vlan-id>" for details
Version=2, Intervals: Query=125, Group Age=260, Max Resp=10, Other Qr=255, Leave Wait=2
VL101: cfg V2, glb cfg active, 5 grp, 5 (*G) cache, no rtr port, 
       My Query address: 10.10.101.1 (configured)
```

**What to look for:**

- `active` or `passive` mode
- Query address (should match your VE interface)
- Number of groups learned

### Display VLAN-Specific Multicast Info

```bash
ICX7150# show ip multicast vlan 101
```

### Display IGMP Group Memberships

```bash
ICX7150# show ip multicast group
```

**Sample Output:**

```
Group Address    VLAN  Port         Type    
239.1.1.1        101   1/1/5        Dynamic  
239.1.1.1        101   1/1/8        Dynamic
239.10.10.1      101   1/1/1-1/1/24 Static
```

### Display Multicast Forwarding Cache (mcache)

```bash
ICX7150# show ip multicast mcache
```

This shows actual data streams being forwarded (not just group memberships).

### Display IGMP Snooping Groups (Alternative)

```bash
ICX7150# show igmp-snooping groups
```

### Display Multicast Traffic Statistics

```bash
ICX7150# show ip multicast traffic
```

Useful for troubleshooting - shows queries sent/received, reports received, etc.

### Check Router Ports

```bash
ICX7150# show ip multicast router-port
```

---

## 5. Common Scenarios and Outcomes

|Scenario|IGMP Snooping|Querier Present|Result|
|---|---|---|---|
|Baseline (proper config)|Enabled|Yes (Active)|✅ Efficient forwarding, minimal bandwidth waste|
|No querier configured|Enabled|No|❌ Tables age out → multicast stops after ~260 sec|
|Snooping disabled|Disabled|N/A|⚠️ Multicast floods to all ports (works but wastes bandwidth)|
|Multiple queriers configured|Enabled|Multiple|✅ Lowest IP wins election, others go passive (auto-resolves)|
|Static groups only|Enabled|Optional|⚠️ Traffic forwarded to static ports only, dynamic joins ignored|
|Multicast router port configured|Enabled|Yes|✅ Multicast can route between VLANs (requires PIM on router)|
|Fast-leave on trunk port|Enabled|Yes|❌ Can break multicast (traffic removed prematurely)|

---

## 6. Frequently Asked Questions

### Q: Can a Layer 2 switch act as an IGMP querier?

**A:** Yes, absolutely. Any Ruckus ICX switch with a VE interface (Layer 3 VLAN interface) on the VLAN can act as a querier in **active mode**. The switch does NOT need to be the default gateway - it just needs a local IP on that VLAN.

### Q: Do other switches in the topology need to know who the querier is?

**A:** No. Other switches automatically **discover** the querier by listening to IGMP query messages. They operate in passive mode and forward queries throughout the VLAN.

### Q: Does the querier send the actual multicast data?

**A:** **No!** The querier only sends **IGMP Query control messages**. The actual multicast data comes from the application server (e.g., your VoIP paging server, IPTV head-end, etc.). The querier's job is purely maintaining group memberships.

### Q: When is a multicast router port required?

**A:** Only when multicast traffic must **cross VLAN boundaries**. For example:

- Paging server in VLAN 100, phones in VLAN 101 → needs router port + PIM routing
- All devices in same VLAN → no router port needed

### Q: What happens if multiple switches are configured as active queriers?

**A:** IGMP has a querier election process. The switch with the **lowest IP address** on the VLAN wins and becomes the active querier. All others automatically become passive. However, it's best practice to configure only ONE switch as active to avoid unnecessary elections.

### Q: Why did my paging system work when I disabled IGMP snooping?

**A:** Disabling snooping causes the switch to **flood all multicast traffic** to every port in the VLAN (like a broadcast). This is inefficient and wastes bandwidth, but it does ensure all devices receive the stream. It's a temporary workaround, not a solution.

### Q: What is the purpose of static-group entries?

**A:** Static groups are for devices that **don't send IGMP join messages** (some legacy paging devices, cameras, etc.). The switch acts as a proxy and tells the querier "I have members for this group" even though no IGMP joins were received. Traffic is then forwarded to the configured static ports.

### Q: Can I run both multicast routing (PIM) and IGMP snooping on the same VLAN?

**A:** **No.** On Ruckus ICX, you cannot enable PIM routing on a VE interface and IGMP snooping on the same VLAN simultaneously. They are mutually exclusive. Choose one:

- **IGMP snooping** for Layer 2 multicast within a VLAN
- **PIM routing** for Layer 3 multicast across VLANs/subnets

### Q: My multicast stopped working after 260 seconds. What happened?

**A:** Your querier is either **down** or **not configured**. Without queries, IGMP group memberships age out (default 260 seconds = 2x query interval + margin). Check:

1. Is there an active querier? (`show ip multicast`)
2. Is the VE interface up? (`show interface ve 101`)
3. Is multicast set to active or passive? (`show ip multicast vlan 101`)

### Q: Should I use IGMPv2 or IGMPv3?

**A:** For most K-12 and SMB deployments with VoIP paging:

- **Use IGMPv2** - simpler, widely supported by all phones
- **Use IGMPv3** only if you need source-specific multicast (SSM) or advanced filtering

IGMPv3 adds source filtering (e.g., "I want group 239.1.1.1 but ONLY from source 10.1.1.100"). Most paging systems don't require this.

### Q: How do I troubleshoot multicast not working?

**Step-by-step checklist:**

1. **Verify snooping is enabled:**
    
    ```bash
    show ip multicast
    ```
    
    Look for "glb cfg active" or "glb cfg passive"
    
2. **Check for an active querier:**
    
    ```bash
    show ip multicast vlan 101
    ```
    
    Look for "My Query address: X.X.X.X" - should show your VE IP
    
3. **Verify group memberships exist:**
    
    ```bash
    show ip multicast group
    ```
    
    You should see entries for your multicast groups
    
4. **Check if traffic is flowing:**
    
    ```bash
    show ip multicast mcache
    ```
    
    Mcache entries appear when actual data traffic flows
    
5. **Look for error counters:**
    
    ```bash
    show ip multicast traffic
    ```
    
    Check for drops, resource exhaustion, etc.
    
6. **Test with flooding (temporary):**
    
    ```bash
    no ip multicast
    ```
    
    If it works now, the issue is with your querier/snooping config
    

### Q: What's the difference between "group" and "mcache"?

**A:**

- **Group table** (`show ip multicast group`): Shows IGMP control plane - who **wants** to receive which groups (based on IGMP joins)
- **Mcache (multicast cache)** (`show ip multicast mcache`): Shows data plane - which multicast streams are **actually being forwarded** through the switch

Both should align. If you see groups but no mcache entries, data isn't flowing.

### Q: Can I see IGMP query messages in packet captures?

**A:** Yes! IGMP queries are sent to `224.0.0.1` (all-hosts multicast). Filter your capture for:

- Destination IP: `224.0.0.1`
- Protocol: IGMP
- Type: Membership Query (0x11)

IGMP Membership Reports are sent to the specific group address (e.g., `239.1.1.1`).

---

## 7. Common Mistakes and Misconceptions

### ❌ Mistake: Enabling fast-leave on all ports

**Problem:** Breaks multicast on trunk ports and ports with multiple devices.  
**Fix:** Only enable fast-leave on access ports with a single endpoint.

### ❌ Mistake: Configuring multiple switches as active queriers

**Problem:** Causes unnecessary querier elections and can confuse debugging.  
**Fix:** Configure only ONE switch per VLAN as active. All others should be passive.

### ❌ Mistake: Disabling snooping to "fix" multicast

**Problem:** Floods multicast everywhere, wastes bandwidth, creates network load.  
**Fix:** Enable snooping and configure a proper querier.

### ❌ Misconception: "The querier sends the multicast traffic"

**Reality:** The querier only sends IGMP control queries. Multicast data comes from your application server.

### ❌ Misconception: "I need PIM for VoIP paging"

**Reality:** Only if paging crosses VLAN boundaries. Single-VLAN paging only needs IGMP snooping.

### ❌ Mistake: Using a device IP as querier address instead of the VE interface

**Problem:** Queries won't be sent from the correct IP, memberships won't be maintained.  
**Fix:** Always use the switch's VE interface IP as the querier address (automatic if VE is configured).

### ❌ Misconception: "224.0.0.x addresses can be controlled with snooping"

**Reality:** The 224.0.0.0/24 range is reserved and always flooded - snooping doesn't apply to these addresses.

---

## 8. Key Takeaways

✅ **IGMP is control plane, multicast is data plane** - they're related but separate.

✅ **Snooping requires an active querier** - without queries, memberships age out and multicast stops.

✅ **Active mode = querier, Passive mode = listener** - only one active querier per VLAN is needed.

✅ **Fast-leave is only safe on access ports** with single devices - never enable on trunks.

✅ **Static groups are for devices that don't send IGMP joins** - use sparingly.

✅ **Flooding (no snooping) works but wastes bandwidth** - querier + snooping is best practice.

✅ **Multicast router ports are only for cross-VLAN routing** - not needed for single-VLAN deployments.

✅ **Always verify with show commands** - check groups, mcache, and traffic statistics regularly.

✅ **You cannot run PIM routing and IGMP snooping on the same VLAN** - they're mutually exclusive.

---

## 9. Additional Resources

- **Ruckus FastIron IP Multicast Configuration Guide** - Official documentation for your firmware version
- **Ruckus Community Forums** - Real-world troubleshooting and configuration examples
- **RFC 2236** - IGMPv2 Protocol Specification
- **RFC 3376** - IGMPv3 Protocol Specification

---

## Document Version

**Version:** 2.0  
**Last Updated:** October 2025  
**Target Firmware:** Ruckus FastIron 08.0.90 and newer  
**Target Audience:** K-12 Networks, Small to Medium Enterprises

---

## Changelog

- **v2.0**: Complete revision with accurate CLI syntax, improved explanations, additional FAQs
- **v1.0**: Initial ChatGPT-generated draft (contained syntax errors and unclear explanations)

---

_This document is provided as-is for educational purposes. Always test configurations in a lab environment before deploying to production. Command syntax may vary between firmware versions - consult official Ruckus documentation for your specific platform._