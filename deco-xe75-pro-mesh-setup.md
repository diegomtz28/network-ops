# TP-Link Deco XE75 Pro Mesh Network — Home Setup & 1GB ISP Upgrade

> **Lab:** apextech.local | **Date:** 2026-03-17 | **Status:** Complete

## Overview

This documents the full home network rebuild using the TP-Link Deco XE75 Pro (3-pack) alongside a new DOCSIS 3.1 1GB ISP service upgrade from 500MB. The Deco replaced the previous setup entirely — no legacy hardware carried over. The mesh serves the home network and acts as the upstream gateway for the apextech.local home lab. Three nodes cover the full house with WiFi 6E tri-band coverage and a dedicated 6GHz backhaul between nodes.

## Environment

**Relevant components for this write-up:**

| Component | Role | Details |
|-----------|------|---------|
| TP-Link Deco XE75 Pro × 3 | Mesh router / gateway | WiFi 6E tri-band, 2.5Gbps WAN port, dedicated 6GHz backhaul |
| ISP DOCSIS 3.1 EMTA | Modem + VoIP | 2.5Gbps LAN port, pre-activated by ISP for 1GB service, no router function |
| Node 1 — Living Room | Main unit / WAN gateway | Connected to modem via ethernet, handles DHCP and routing |
| Node 2 — Diego's Room | Satellite mesh node | Wireless backhaul over 6GHz |
| Node 3 — Brother's Room | Satellite mesh node | Wireless backhaul over 6GHz |
| Wired PC | Lab / primary desktop | Connected via ethernet to Deco LAN port |
| MacBook Pro M1 2021 | Personal laptop | WiFi 6 (not 6E) — 5GHz max |
| Proxmox Host (HP Z2450) | Lab hypervisor | [TODO: confirm wired or wireless connection] |

## Steps Taken

1. **Confirmed modem requires no configuration**
   The ISP-provided DOCSIS 3.1 EMTA came pre-activated for the new 1GB service. Since it is a modem-only device (EMTA — no built-in router or WiFi), bridge mode was not needed. The Deco receives the public IP directly with no double-NAT.

2. **Connected Node 1 (main unit) to the modem**
   Ran an ethernet cable from the modem's LAN port into the Deco XE75 Pro WAN port. Powered on Node 1 and waited for the LED to pulse, indicating it was ready for setup.

3. **Installed the TP-Link Deco app and created a TP-Link account**
   Downloaded the Deco app, created a TP-Link cloud account, and initiated setup by selecting Add Deco → XE75 Pro.

4. **Configured internet connection type**
   Selected Dynamic IP — ISP uses DHCP, no PPPoE credentials required. The Deco pulled a public IP from the modem successfully.

5. **Set WiFi SSID and password**
   Configured a unified SSID across all bands (2.4GHz / 5GHz / 6GHz). Devices auto-connect to the best available band. Password set with WPA3.

6. **Added satellite nodes**
   Powered on Node 2 (Diego's room) and Node 3 (brother's room). Added both through the app. Each node backhauled wirelessly over the dedicated 6GHz band, keeping 2.4GHz and 5GHz fully available for client devices.

7. **Updated firmware on all three nodes**
   Navigated to More → Deco Network → each unit → checked for and applied firmware updates before putting the network into production.

8. **Configured guest network**
   Created a separate guest SSID for guest devices and IoT. Keeps IoT traffic isolated from the main network and the lab.

9. **Set DNS to Cloudflare/Google**
   More → Advanced → Internet → DNS → set to `1.1.1.1` (primary) and `8.8.8.8` (secondary) for faster resolution and reduced ISP DNS latency.

10. **Reserved static IPs for lab machines via DHCP reservation**
    More → Advanced → DHCP → Address Reservation. Assigned fixed IPs to lab machines so apextech.local infrastructure has consistent addresses across reboots.
    [TODO: document the actual IP reservations — which machine got which IP]

11. **Configured port forwarding for lab access**
    More → Advanced → Port Forwarding. Added rules to expose lab services externally, pointed at the reserved IPs from Step 10.
    [TODO: document specific port forwarding rules configured]

12. **Enabled Band Steering**
    More → Advanced → Wireless → Band Steering confirmed on. Pushes capable devices toward higher bands automatically.

## Why These Decisions Were Made

- **EMTA modem, no bridge mode needed:** The ISP-provided modem is a modem-only device — it passes through the public IP directly to the Deco. No router functionality to bypass, no double-NAT to worry about.

- **2.5Gbps modem → 2.5Gbps Deco WAN port:** The DOCSIS 3.1 EMTA's 2.5Gbps LAN port feeds directly into the XE75 Pro's 2.5Gbps WAN port. This gives full 1GB throughput with headroom for future ISP upgrades.

- **Dedicated 6GHz backhaul:** The XE75 Pro reserves the 6GHz band for node-to-node communication. This is why WiFi speeds don't degrade across hops — client bands aren't shared with backhaul traffic.

- **Node in each major room:** Living room (central/modem location), Diego's room, and brother's room gives full house coverage without relying on signal bleed from a single unit.

- **DNS set to 1.1.1.1/8.8.8.8:** ISP default DNS is often slow. Cloudflare's 1.1.1.1 is consistently faster and doesn't log queries.

- **MacBook Pro M1 2021 on 5GHz:** The M1 MacBook Pro supports WiFi 6 but not WiFi 6E — it cannot connect to the 6GHz band. 300-600 Mbps on 5GHz is expected and normal for this hardware. WiFi 6E support on MacBooks began with the M3 (2023).

## Test & Validation

- **Test:** Wired throughput to ISP
  **Method:** Laptop connected via ethernet to Deco LAN port, ran speed test at fast.com
  **Result:** ~900 Mbps down — expected, overhead on 1GB plan accounts for the difference

- **Test:** WiFi throughput — MacBook Pro M1 2021
  **Method:** Speed test from MacBook on 5GHz band
  **Result:** 300-600 Mbps — normal for WiFi 6 on 5GHz, hardware cannot use 6GHz

- **Test:** Wireless Diagnostics channel scan
  **Method:** Option + click WiFi icon → Open Wireless Diagnostics → Window → Scan
  **Result:** [TODO: document channel recommendations and any interference found]

- **Test:** Signal strength (RSSI) per node
  **Method:** Option + click WiFi icon on MacBook → check RSSI value
  **Result:** [TODO: document RSSI reading per room]

## Notes & Next Steps

- **6GHz optimization:** Any WiFi 6E capable device (iPhone 15+, M3 MacBook, WiFi 6E laptops) will see significantly higher speeds — 700-900 Mbps is achievable close to any node.
- **Lab network segmentation:** Long-term, consider putting apextech.local lab traffic on its own VLAN behind pfSense rather than sharing the flat home network. The Deco supports VLANs via its managed switch ports.
- **QoS:** More → Advanced → QoS — if lab work or gaming competes with other household traffic, enable QoS and prioritize the wired PC.
- **Node placement tuning:** If speeds in brother's room or Diego's room are consistently low, adjust node position — 6GHz backhaul range is shorter than 5GHz. Nodes should ideally be within 30-40 feet of each other.
