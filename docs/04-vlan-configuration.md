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
