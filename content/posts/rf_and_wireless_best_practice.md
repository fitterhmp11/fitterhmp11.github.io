---
title: "RF and Wireless Networks: Operations and Best Practices"
date: 2025-11-21T02:00:00
draft: false
tags: [ "wireless", "aruba", "troubleshooting"]
categories: 
 - Networking
  url: "rf-and-wireless"
summary: "Information gathered while troubleshooting Wireless issues"
---

## 1. Introduction

This document provides an engineering-focused overview of RF behavior, client roaming, and Aruba WLAN best practices. It explains why wireless clients may experience performance issues or roam poorly and provides practical guidance for designing robust Wi-Fi networks.

---

## 2. RF Fundamentals

Wireless network performance is heavily influenced by RF properties.

### 2.1 Signal and SNR
- **RSSI (Signal Strength)**: Determines if a client can decode frames reliably.
- **Noise Floor**: Baseline RF energy from other devices.
- **SNR (Signal-to-Noise Ratio)**: Key for determining achievable data rates.
- **Best Practice**: Design for ≥ -65 dBm RSSI (minimum target; -67 dBm is common design threshold) and ≥ 25 dB SNR for consistent performance. Higher-density environments benefit from targeting -65 dBm or better.

### 2.2 Band Selection
- **2.4 GHz**: Longer range, limited capacity, more interference.
- **5 GHz**: Higher throughput, better client density handling.
- **6 GHz**: Clean spectrum, requires modern Wi-Fi 6E clients and typically WPA3 security.
- **Best Practice**: Encourage 5 GHz connections with band steering; limit 2.4 GHz coverage.

### 2.3 Channel Width
- **20 MHz**: Optimal for high-density enterprise deployments.
- **40/80 MHz**: Use only in low-density or low-interference environments.

---

## 3. Aruba Wireless Configuration Considerations

### 3.1 Transmit Power Control
- High 2.4 GHz TX power → clients stick to far APs.
- Low 2.4/5 GHz power → smoother roaming and less co-channel interference.
- **Best Practice**: 2.4 GHz at 9–12 dBm, 5 GHz at 12–18 dBm (as maximum values; ARM/TPC may adjust lower dynamically).

### 3.2 Airtime Fairness
- **Default**: Balanced approach.
- **Fair Access**: Equalizes airtime for all clients.
- **Preferred Access**: Prioritizes fast clients.
- **Best Practice**: Use Default or Fair Access for typical enterprise environments.

### 3.3 Broadcast Filtering
- **ARP Filtering**: AP proxies ARP to reduce broadcast.
- **Unicast ARP Only**: Converts ARP broadcasts to unicast frames sent to each associated client.
- **All**: Filters all broadcast/multicast frames (may break DHCP discovery in some configurations).
- **Disabled**: No filtering.
- **Best Practice**: Use ARP or Unicast ARP Only to optimize airtime without breaking discovery.

### 3.4 DFS Channels
- Dynamic Frequency Selection channels offer cleaner spectrum.
- Some legacy clients may avoid DFS channels.
- **Best Practice**: Enable DFS unless known client issues exist.

---

## 4. Fast Roaming Protocols

### 4.1 802.11r (Fast BSS Transition)
- Reduces handshake latency during AP transitions.
- Effective for PSK and enterprise networks with modern clients.

### 4.2 802.11k (Neighbor Reports)
- Provides clients with nearby AP information for faster scanning.
- Low-risk, universally recommended.

### 4.3 802.11v (BSS Transition Management)
- Allows APs to suggest better APs for clients.
- Many clients ignore BSS Transition Management frames, especially iOS devices.

**Best Practice**: Enable 11k + 11v universally; enable 11r after validating client support.

---

## 5. Common Client Issues

### 5.1 Stuck on 2.4 GHz

**Causes:**
- Strong RSSI from 2.4 GHz APs.
- Client preference algorithms.
- High 2.4 GHz transmit power.

**Mitigation**: Lower 2.4 GHz power, use band steering, ensure consistent 5 GHz coverage.

### 5.2 Slow or Failed Roams
- Caused by insufficient RSSI/SNR, airtime congestion, or delayed DHCP responses (when roaming between subnets).
- **Best Practice**: Optimize RF coverage, enable fast roaming protocols, and monitor client performance.

---

## 6. Roaming Timing Considerations
- **Probe/Auth/Assoc**: ~10 ms
- **4-Way Handshake**: 10–40 ms
- **802.11r FT handshake**: 5–10 ms
- **DHCP (if roaming to new subnet)**: 200–800 ms

**Note**: Same-subnet roaming does not require DHCP, reducing total roam time significantly.

**Goal**: Keep total roam latency <50 ms for real-time applications.

---

## 7. Troubleshooting Workflow
1. **Survey RF Coverage**: Verify RSSI and SNR levels.
2. **Check AP and Channel Settings**: Validate TX power, DFS usage, airtime fairness.
3. **Review Client Logs**: Identify stuck clients or slow roam events.
4. **Monitor Aruba Central Metrics**: Observe client distribution, retries, and failed roam attempts.
5. **Adjust Settings Iteratively**: Tune band steering, power levels, and roaming protocols as needed.

---

## 8. Summary of Best Practices
- Design for ≥ -65 dBm RSSI (minimum; -67 dBm for general coverage) and ≥ 25 dB SNR.
- Prioritize 5 GHz, limit 2.4 GHz power.
- Use 20 MHz channels for high-density environments.
- Enable 11k + 11v; enable 11r selectively after validation.
- Apply ARP filtering or Unicast ARP Only.
- Monitor client behavior and adjust RF settings iteratively.

This guide consolidates enterprise RF and Aruba-specific WLAN best practices, providing a reference for design, troubleshooting, and optimization in Wi-Fi networks.