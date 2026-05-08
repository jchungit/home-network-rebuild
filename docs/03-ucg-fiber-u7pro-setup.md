# 03 — UCG-Fiber & U7 Pro Deployment

## Objective

Replace the consumer-grade Netgear RAXE300 with enterprise-grade Ubiquiti UniFi hardware to support VLAN segmentation, centralized network management, and a scalable homelab infrastructure.

---

## Hardware Decision

### Why Upgrade from RAXE300?

The Netgear RAXE300 is a capable consumer WiFi 6E router, but it has a critical limitation for this use case:

- **Not VLAN-aware in a way that integrates with enterprise management**
- Cannot create per-SSID (Service Set Identifier) VLAN assignments
- No centralized dashboard for traffic monitoring
- Limited firewall rule granularity

For homelab and cybersecurity work, VLAN segmentation is essential — especially to isolate IoT (Internet of Things) devices, contain homelab traffic, and protect personal devices.

### Hardware Selected

| Device | Model | Price | Role |
|--------|-------|-------|------|
| UniFi Cloud Gateway Fiber | UCG-Fiber | ~$199 | Main router + firewall + UniFi controller |
| UniFi WiFi 7 AP | U7 Pro | ~$179 | VLAN-aware wireless access point |
| **Bundle (Microcenter)** | UCG-Fiber + U7 Pro | **$399.98** | Both above |

### Why UCG-Fiber Over UCG-Max

The UCG-Fiber offers an **SFP+ (Small Form-factor Pluggable Plus)** port in addition to the standard RJ45 WAN port. This enables a future upgrade path:

```
Phase 1 (Current):
BGW320 (IP Passthrough) → Ethernet → UCG-Fiber RJ45 WAN port

Phase 2 (Future):
SFP+ fiber module → UCG-Fiber SFP+ WAN port → BGW320 completely bypassed
```

The SFP+ port allows direct fiber connection to AT&T's ONT, eliminating the BGW320 entirely once BGW bypass authentication is configured.

---

## Physical Installation

### Network Topology (3-Floor Home)

```
3rd Floor — BGW320-500 + UCG-Fiber
2nd Floor — Netgear RAXE300 (retained as secondary AP)
1st Floor — U7 Pro (ceiling mount, PoE-powered)
```

### U7 Pro Power Requirement — PoE+

The U7 Pro has **no power adapter** — it is powered entirely through the ethernet cable using **PoE+ (Power over Ethernet Plus, 802.3at standard)**. The UCG-Fiber's LAN ports are not PoE, so a **PoE injector** was used:

```
UCG-Fiber LAN port → Ethernet → PoE Injector → Ethernet → U7 Pro
                                     │
                               Power outlet
```

A PoE injector plugs inline between the switch/router and the AP, injecting power over the same cable that carries data.

### U7 Pro Placement

Ceiling-mounted on the 1st floor, centrally located. The U7 Pro broadcasts in a dome pattern downward and outward — ceiling center is the optimal placement for maximum coverage radius.

---

## UniFi Initial Setup

### Step 1 — Connect Hardware
1. Plug ethernet from BGW320 LAN port into UCG-Fiber **RJ45 WAN port**
2. Plug ethernet from UCG-Fiber LAN port → PoE injector → U7 Pro
3. Power on UCG-Fiber

### Step 2 — Access UniFi Setup Wizard
1. Connect a device to the UCG-Fiber via ethernet or its default WiFi network
2. Navigate to `https://192.168.1.1` or `unifi.ui.com`
3. Follow the setup wizard

### Step 3 — Adopt the U7 Pro
Once the UCG-Fiber is online, the U7 Pro appears automatically in the UniFi console as a **pending device**:
1. In UniFi console → **Devices**
2. Click on U7 Pro → **Adopt**
3. Wait for adoption and provisioning (2–3 minutes)
4. U7 Pro status changes to **Connected** ✅

### Step 4 — Update Firmware
Before configuring anything:
1. In UniFi console → **Devices**
2. Click each device → **Settings** → **Update**
3. Update both UCG-Fiber and U7 Pro to latest firmware
4. Allow devices to reboot

---

## Network State After This Phase

```
AT&T Fiber (300 Mbps)
      │
      ▼
BGW320-500 (IP Passthrough)
  192.168.1.254
      │ Ethernet — WAN
      ▼
UCG-Fiber (Main Router + Firewall + UniFi Controller)
  192.168.1.1
      │ LAN (via PoE injector)
      ▼
U7 Pro (WiFi 7 AP — 1st floor ceiling)
  Managed by UCG-Fiber UniFi console
```

All devices now visible and manageable from a single UniFi dashboard. ✅

---

**Next Step:** [04 — VLAN Configuration](04-vlan-configuration.md)
