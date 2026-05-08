# 🏠 Home Network Rebuild — From BGW320 Diagnosis to UniFi VLAN Segmentation

> **A real-world homelab project documenting the full journey of diagnosing a residential network, replacing consumer hardware with enterprise-grade equipment, and building a segmented VLAN network for Personal, Homelab, and IoT traffic.**

---

## 📋 Project Overview

This project documents the complete rebuild of a 3-floor home network — starting from diagnosing slow speeds and a double NAT (Network Address Translation) issue on an AT&T BGW320-500 gateway, all the way through deploying a Ubiquiti UCG-Fiber with a U7 Pro access point and configuring VLAN segmentation for three distinct network zones.

**Skills demonstrated:**
- Network troubleshooting and diagnostics
- ISP gateway configuration (IP Passthrough)
- Enterprise network hardware deployment (Ubiquiti UniFi)
- VLAN design and segmentation
- Wireless network architecture
- Network security (IoT isolation, homelab containment)

---

## 🗂️ Documentation Index

| Doc | Description |
|-----|-------------|
| [01 — BGW320 Diagnosis](docs/01-bgw320-diagnosis.md) | Identifying the double NAT problem and slow speeds |
| [02 — IP Passthrough Setup](docs/02-ip-passthrough-setup.md) | Configuring AT&T gateway to pass traffic to RAXE300 |
| [03 — UCG-Fiber & U7 Pro Setup](docs/03-ucg-fiber-u7pro-setup.md) | Replacing consumer hardware with UniFi stack |
| [04 — VLAN Configuration](docs/04-vlan-configuration.md) | Building Personal, Homelab, and IoT network segments |

---

## 🏗️ Final Network Architecture

```
Internet (AT&T Fiber 300 Mbps)
          │
          ▼
┌─────────────────────┐
│   BGW320-500        │  3rd Floor
│   AT&T Gateway      │  (IP Passthrough mode — modem only)
│   192.168.1.254     │
└────────┬────────────┘
         │ Ethernet (WAN)
         ▼
┌─────────────────────┐
│   UCG-Fiber         │  3rd Floor
│   UniFi Gateway     │  (Main router, firewall, VLAN routing)
│   192.168.1.1       │
└────────┬────────────┘
         │ Ethernet (LAN)
         ▼
┌─────────────────────┐
│   U7 Pro            │  1st Floor (ceiling mount)
│   UniFi WiFi 7 AP   │  (VLAN-aware access point)
│   PoE powered       │
└─────────────────────┘
```

---

## 🌐 VLAN Network Map

| VLAN | ID | Subnet | Purpose |
|------|----|--------|---------|
| Personal | 10 | 192.168.10.0/24 | Daily use devices — MacBook, iPhone, iPad |
| Homelab | 20 | 192.168.20.0/24 | ThinkPad P1, VMs, Wazuh SIEM, Kali Linux |
| IoT | 30 | 192.168.30.0/24 | Smart TVs, speakers, cameras, smart home |

---

## 🧰 Hardware Used

| Device | Role |
|--------|------|
| AT&T BGW320-500 | ISP-provided fiber gateway (ONT + modem) |
| Ubiquiti UCG-Fiber | Main router, firewall, UniFi controller |
| Ubiquiti U7 Pro | WiFi 7 access point (PoE+) |
| Netgear RAXE300 | Retired to spare / secondary AP (optional) |

---

## 📅 Timeline

1. **Phase 1** — Diagnosed double NAT issue causing speed bottleneck
2. **Phase 2** — Configured IP Passthrough on BGW320 to eliminate double NAT
3. **Phase 3** — Purchased and deployed UCG-Fiber + U7 Pro bundle ($399.98 from Microcenter)
4. **Phase 4** — Configured VLANs and wireless SSIDs for network segmentation
5. **Phase 5** — Validated isolation between network segments

---

## 💡 Why This Matters (Security Perspective)

VLAN segmentation follows the **principle of least privilege** at the network layer:

- **IoT devices** (smart TVs, cameras) are isolated — if one is compromised (like in the Mirai botnet attack), it cannot pivot to personal or homelab devices
- **Homelab VMs** running tools like Wazuh SIEM (Security Information and Event Management) and Kali Linux are contained — offensive tools cannot accidentally reach personal devices
- **Personal devices** operate in a clean network with no exposure to potentially vulnerable IoT hardware

This is the same segmentation strategy used in enterprise environments — implemented here at home scale.

---

*Built by Jin Chung | IT Support & Cybersecurity | Anaheim, CA*
*Fullstack Academy Cybersecurity Analytics Bootcamp | CompTIA A+ / Security+ candidate*
