[01-bgw320-diagnosis.md](https://github.com/user-attachments/files/27508304/01-bgw320-diagnosis.md)
# 01 — BGW320-500 Diagnosis: Identifying the Double NAT Problem

## Problem Statement

After setting up AT&T Fiber (300 Mbps plan) with a Netgear RAXE300 router, speed tests were returning significantly below expected throughput:

| Device | Connection | Speed Test Result |
|--------|------------|-------------------|
| ASUS ROG Desktop | Wired Ethernet | ~142 Mbps |
| MacBook | WiFi | ~179 Mbps |
| iPhone | WiFi | ~181 Mbps |

**Expected:** ~280–300 Mbps on wired, ~250–290 Mbps on WiFi close to router  
**Actual:** Well below expected on all devices

---

## Root Cause Analysis

### The Double NAT (Network Address Translation) Problem

Running `ipconfig` on the Windows desktop revealed the issue:

```
Default Gateway: 192.168.1.254  ← AT&T BGW320 gateway IP
```

The network topology at the time looked like this:

```
AT&T Fiber (300 Mbps)
      │
      ▼
BGW320-500 Gateway (3rd floor)     ← Acting as ROUTER #1
  192.168.1.254
  DHCP range: 192.168.1.x
      │
      │ Ethernet (through wall)
      ▼
RAXE300 Router (1st floor)         ← Acting as ROUTER #2 behind Router #1
  192.168.1.1
  DHCP range: 192.168.1.x
      │
      ├── Desktop (wired)
      ├── MacBook (WiFi)
      └── iPhone (WiFi)
```

**The problem:** Both the BGW320 and the RAXE300 were acting as routers simultaneously. This is called **Double NAT** — traffic had to be translated twice (once by AT&T's gateway, again by the RAXE300) before reaching the internet. This creates:

- Latency overhead on every packet
- Throughput bottleneck
- Port forwarding complexity[Uploading 02-ip-passthrough-setup.md…]()

- Network isolation between devices on different routers

### Why the BGW320 Causes This

The AT&T BGW320-500 is a **combo device** — it functions as:
1. An **ONT (Optical Network Terminal)** — converts fiber light signals to ethernet
2. A **router** — handles NAT and DHCP by default

AT&T requires the BGW320 to stay connected (it handles authentication with AT&T's fiber network), but it doesn't have to act as the primary router.

---

## Diagnostic Steps Taken

1. **Speed test from wired desktop** → 142 Mbps (well below 300 Mbps plan)
2. **Speed test from WiFi devices** → ~180 Mbps
3. **Ran `ipconfig`** on desktop → Default Gateway showed `192.168.1.254` (AT&T gateway, not RAXE300)
4. **Logged into AT&T gateway** at `http://192.168.1.254` → Confirmed it was routing traffic
5. **Confirmed RAXE300** was also routing → Double NAT confirmed

---

## Key Learnings

- The BGW320-500 is a fiber ONT + router combo — it cannot be put in true bridge mode by AT&T policy
- **IP Passthrough** is AT&T's equivalent of bridge mode — it passes the public IP directly to a designated device (the RAXE300)
- Without IP Passthrough, any router placed behind the BGW320 creates Double NAT

---

**Next Step:** [02 — IP Passthrough Setup](02-ip-passthrough-setup.md)

# 02 — IP Passthrough Setup on AT&T BGW320-500

## Objective

Configure the AT&T BGW320-500 to operate in **IP Passthrough mode**, effectively making it a transparent modem/ONT so the RAXE300 (and later the UCG-Fiber) becomes the true router for the network.

---

## What Is IP Passthrough?

IP Passthrough is AT&T's version of bridge mode. Instead of the BGW320 routing traffic and assigning private IPs, it passes the **public WAN (Wide Area Network) IP** directly to a designated downstream device.

```
Before IP Passthrough:
Internet → BGW320 (routes) → RAXE300 (routes again) → Devices
           [Double NAT]

After IP Passthrough:
Internet → BGW320 (passes through) → RAXE300 (routes) → Devices
           [Single NAT — clean]
```

---

## Step-by-Step Configuration

### Step 1 — Access the BGW320 Admin Interface
1. Open a browser and navigate to: `http://192.168.1.254`
2. Log in with:
   - **Username:** `admin`
   - **Password:** Found on the sticker on the side/bottom of the BGW320

### Step 2 — Navigate to IP Passthrough Settings
1. Click **Firewall** in the top navigation
2. Click **IP Passthrough** in the left sidebar

### Step 3 — Configure the Settings

| Setting | Value |
|---------|-------|
| Allocation Mode | **Passthrough** |
| Passthrough Mode | **DHCPS-fixed** |
| Default Server Internal Address | RAXE300's IP (e.g., `192.168.1.1`) |
| Passthrough Fixed MAC Address | RAXE300's MAC address (from sticker on bottom of RAXE300) |

> **Note:** The RAXE300's MAC address was located on the label on the bottom of the device under "MAC Address" or "Router MAC."

### Step 4 — Save and Reboot

1. Click **Save**
2. Reboot the **BGW320 first** — wait 2 minutes for it to fully come online
3. Then reboot the **RAXE300** — wait 2 minutes
4. Reconnect all devices

> **Reboot order matters:** Always reboot the modem/gateway first. The router needs the gateway to be fully online before it can receive its IP address.

---

## Verification

After rebooting, ran `ipconfig` on the desktop:

```
Default Gateway: 192.168.1.1   ← Now showing RAXE300, not AT&T gateway ✅
```

Speed test results after IP Passthrough:

| Device | Before | After |
|--------|--------|-------|
| Wired Desktop | ~142 Mbps | ~285 Mbps |
| MacBook WiFi | ~179 Mbps | ~260 Mbps |
| iPhone WiFi | ~181 Mbps | ~255 Mbps |

**Result:** Speeds jumped significantly across all devices — close to the 300 Mbps plan. ✅

---

## Network State After This Phase

```
AT&T Fiber (300 Mbps)
      │
      ▼
BGW320-500 (IP Passthrough — modem only)
  192.168.1.254  ← Still reachable for admin purposes
      │
      ▼
RAXE300 in AP mode (1st floor)
  192.168.1.1  ← Now the true router
      │
      ├── ASUS ROG Desktop (wired)
      ├── MacBook (WiFi)
      └── All other devices
```

> **Note:** The RAXE300 was eventually moved to AP (Access Point) mode and placed on the 2nd floor for better whole-home coverage while the BGW320 handled routing through IP Passthrough.

---

**Next Step:** [03 — UCG-Fiber & U7 Pro Setup](03-ucg-fiber-u7pro-setup.md)

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

# 04 — VLAN Configuration: Personal, Homelab, and IoT Segmentation

## Objective

Design and implement three isolated network segments using VLANs (Virtual Local Area Networks) managed through the UCG-Fiber's UniFi console. Each segment serves a distinct security purpose and contains traffic that should not cross between zones.

---

## Why VLAN Segmentation?

This follows the **principle of least privilege at the network layer** — the same concept used in enterprise environments to contain breaches and limit lateral movement.

### Real-World Threat Context — The Mirai Botnet (2016)

The Mirai botnet attack generated record-breaking DDoS (Distributed Denial of Service) traffic by compromising IoT devices using a list of 61 default username/password combinations. Compromised devices included IP cameras, DVRs, routers, and baby monitors. The attack took down Twitter, Netflix, Reddit, and GitHub.

**VLAN segmentation directly mitigates this threat:** Even if an IoT device is compromised, it cannot reach personal computers or homelab servers — it is trapped in its own isolated network segment.

---

## VLAN Design

| VLAN Name | VLAN ID | Subnet | Gateway | Purpose |
|-----------|---------|--------|---------|---------|
| Personal | 10 | 192.168.10.0/24 | 192.168.10.1 | Daily use: MacBook, iPhone, iPad |
| Homelab | 20 | 192.168.20.0/24 | 192.168.20.1 | ThinkPad P1, VMs, Wazuh SIEM, Kali Linux |
| IoT | 30 | 192.168.30.0/24 | 192.168.30.1 | Smart TVs, speakers, cameras, smart home |

---

## Configuration in UniFi Console

### Step 1 — Create VLANs (Networks)

In UniFi Console → **Settings** → **Networks** → **Add New Network**:

**Personal VLAN:**
- Name: `Personal`
- VLAN ID: `10`
- Gateway IP: `192.168.10.1/24`
- DHCP: Enabled
- DHCP Range: `192.168.10.100 – 192.168.10.254`

**Homelab VLAN:**
- Name: `Homelab`
- VLAN ID: `20`
- Gateway IP: `192.168.20.1/24`
- DHCP: Enabled
- DHCP Range: `192.168.20.100 – 192.168.20.254`

**IoT VLAN:**
- Name: `IoT`
- VLAN ID: `30`
- Gateway IP: `192.168.30.1/24`
- DHCP: Enabled
- DHCP Range: `192.168.30.100 – 192.168.30.254`

---

### Step 2 — Create Wireless SSIDs (WiFi Networks)

In UniFi Console → **Settings** → **WiFi** → **Add New WiFi Network**:

Create one SSID per VLAN, each mapped to its respective network:

| SSID Name | Network | Band | Notes |
|-----------|---------|------|-------|
| `HomeNetwork` | Personal (VLAN 10) | 2.4 + 5 + 6 GHz | Main personal SSID |
| `Homelab` | Homelab (VLAN 20) | 5 GHz | ThinkPad P1, lab devices |
| `IoT-Devices` | IoT (VLAN 30) | 2.4 GHz | Smart home devices |

The U7 Pro broadcasts all SSIDs simultaneously, each tagged to its respective VLAN. Clients connecting to `IoT-Devices` automatically receive a `192.168.30.x` IP and are isolated from devices on `HomeNetwork`.

---

### Step 3 — Firewall Rules

In UniFi Console → **Settings** → **Firewall & Security** → **Firewall Rules**:

**Block IoT → Personal:**
- Action: Drop
- Source: IoT network (192.168.30.0/24)
- Destination: Personal network (192.168.10.0/24)

**Block IoT → Homelab:**
- Action: Drop
- Source: IoT network (192.168.30.0/24)
- Destination: Homelab network (192.168.20.0/24)

**Block Homelab → Personal (optional — tighten for offensive tool isolation):**
- Action: Drop
- Source: Homelab network (192.168.20.0/24)
- Destination: Personal network (192.168.10.0/24)

**Allow all → Internet:**
- All VLANs maintain internet access
- Inter-VLAN routing is blocked, but WAN (Wide Area Network) access is permitted

---

### Step 4 — Homelab VM Integration

The ThinkPad P1 Gen 6 runs VirtualBox VMs (Wazuh SIEM, Kali Linux, Windows Server 2022) for cybersecurity homelab work.

To get individual VMs visible on the Homelab VLAN with their own IPs:

**Option A — USB-C to Ethernet Adapter (Full VLAN control):**
1. Connect Anker USB-C Ethernet adapter to ThinkPad P1
2. Run ethernet cable to a wall jack on the Homelab VLAN
3. In VirtualBox → VM Settings → Network → Adapter → **Bridged Adapter** → select USB ethernet interface
4. Each VM receives its own `192.168.20.x` IP from UCG-Fiber DHCP
5. UCG-Fiber sees every VM individually — full firewall rules and monitoring apply

**Option B — VirtualBox NAT Network (software-only):**
- VMs share the ThinkPad's IP, hidden behind VirtualBox NAT
- UCG-Fiber only sees the ThinkPad — individual VMs are invisible
- Simpler but no per-VM firewall control or traffic visibility

> Option A is strongly preferred for a cybersecurity homelab — full visibility is essential for Wazuh SIEM monitoring and interview-ready documentation.

---

## Final Segmented Network Architecture

```
Internet
    │
BGW320 (IP Passthrough)
    │
UCG-Fiber
    ├── VLAN 10 — Personal (192.168.10.0/24)
    │       ├── MacBook Air M1
    │       ├── iPhone
    │       └── iPad
    │
    ├── VLAN 20 — Homelab (192.168.20.0/24)
    │       ├── ThinkPad P1 Gen 6
    │       ├── VM: Wazuh SIEM (agent: thinkpad-p1)
    │       ├── VM: Kali Linux
    │       └── VM: Windows Server 2022 (Domain Controller)
    │
    └── VLAN 30 — IoT (192.168.30.0/24)
            ├── Smart TV
            ├── Smart speakers
            └── Other smart home devices

ASUS ROG Desktop (wired) → Personal VLAN 10

Firewall Rules:
  IoT ──✗──► Personal   (blocked)
  IoT ──✗──► Homelab    (blocked)
  All ──✓──► Internet   (allowed)
```

---

## Security Benefits Achieved

- **IoT isolation** — compromised smart device cannot reach personal computers or homelab
- **Homelab containment** — offensive security tools (Kali, Metasploit) cannot accidentally reach personal devices
- **Traffic visibility** — UCG-Fiber logs all inter-VLAN traffic attempts for monitoring in Wazuh
- **Enterprise-equivalent segmentation** — same architecture used in corporate environments, built and operated at home scale

---

*This VLAN setup is directly applicable to enterprise networking concepts tested in CompTIA A+ (Network+ concepts), Security+, and relevant to SOC (Security Operations Center) Analyst, Help Desk, and Network Administrator roles.*


