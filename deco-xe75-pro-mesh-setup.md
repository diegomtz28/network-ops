# TP-Link Deco XE75 Pro — Home Mesh Network Rebuild

![Status](https://img.shields.io/badge/status-complete-brightgreen)
![WiFi](https://img.shields.io/badge/WiFi-6E-blue)
![Speed](https://img.shields.io/badge/ISP-1Gbps-orange)
![Nodes](https://img.shields.io/badge/nodes-3--pack-purple)

> Full home network rebuild. Upgraded ISP from 500MB → 1GB, deployed a 3-node WiFi 6E mesh, and connected it to my home lab (apextech.local).

---

## The Gear

| | Hardware | Key Spec |
|---|----------|----------|
| 📡 | TP-Link Deco XE75 Pro × 3 | WiFi 6E, 2.5Gbps WAN, dedicated 6GHz backhaul |
| 🔌 | DOCSIS 3.1 EMTA | 2.5Gbps LAN out, modem-only (no router) |
| 💻 | MacBook Pro M1 2021 | WiFi 6 client (5GHz) |
| 🖥️ | Wired PC | Ethernet → Deco LAN |
| 🧪 | Proxmox Host HP Z2450 | apextech.local lab hypervisor |

---

## Network Topology

```
Internet (1Gbps)
       │
  [DOCSIS 3.1 EMTA]        ← modem only, no routing, no double-NAT
       │ ethernet
  [Deco XE75 Pro]           ← Node 1 — Living Room (main unit / gateway)
   /           \
  6GHz        6GHz          ← dedicated backhaul band (not shared with clients)
  /               \
[Node 2]        [Node 3]
Diego's Room    Brother's Room
```

---

## Speed Results

| Connection | Speed | Notes |
|------------|-------|-------|
| Wired (ethernet) | ~900 Mbps | ISP overhead accounts for ~10% |
| WiFi 5GHz (MacBook M1) | 300–600 Mbps | M1 is WiFi 6, not 6E — 5GHz ceiling |
| WiFi 6GHz (6E device) | 700–900 Mbps | Available for WiFi 6E capable hardware |

---

## What I Configured

**Internet**
- Connection type: Dynamic IP (ISP uses DHCP, no PPPoE needed)
- DNS: `1.1.1.1` / `8.8.8.8` — Cloudflare + Google over ISP default

**WiFi**
- Unified SSID across 2.4 / 5 / 6GHz — devices auto-select best band
- WPA3 security
- Band steering enabled — pushes devices to higher bands automatically
- Guest network — isolated SSID for guests and IoT devices

**Lab Integration**
- DHCP reservations for all apextech.local machines (consistent IPs across reboots)
- Port forwarding rules for lab service access
- All three nodes firmware updated before going live

---

## Why the XE75 Pro

The key feature is the **dedicated 6GHz backhaul** — the nodes talk to each other on 6GHz exclusively, so the 2.4 and 5GHz bands are completely free for client devices. Older mesh systems split backhaul across the same bands clients use, which tanks speeds the further you get from the main unit. This doesn't have that problem.

The **2.5Gbps WAN port** also matches the modem's 2.5Gbps output — no bottleneck at the handoff even as ISP speeds grow.

---

## Next Steps

- [ ] Add VLAN segmentation — put lab traffic on its own subnet behind pfSense
- [ ] Document final IP reservation table
- [ ] Enable QoS and prioritize wired PC for lab work
- [ ] Run RSSI per-room baseline and log it
