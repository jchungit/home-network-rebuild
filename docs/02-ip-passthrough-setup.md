# 02 — IP Passthrough Setup on AT&T BGW320-500

## Objective

With the faulty BGW320-500 replaced by AT&T with a new stable unit, the next step was configuring **IP Passthrough mode** — effectively making the BGW320 a transparent modem/ONT so the RAXE300 (and later the UCG-Fiber) becomes the true router for the network.

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

Speed test results after new modem + IP Passthrough:

| Device | Faulty Modem (throttled) | New Modem + IP Passthrough |
|--------|--------------------------|----------------------------|
| Wired Desktop | ~142–150 Mbps | ~285–295 Mbps stable |
| MacBook WiFi | ~179 Mbps | ~260 Mbps stable |
| iPhone WiFi | ~181 Mbps | ~255 Mbps stable |

**Result:** Speeds stabilized at near-plan rates across all devices with no throttling. ✅

---

## Network State After This Phase

```
AT&T Fiber (300 Mbps)
      │
      ▼
BGW320-500 NEW UNIT (IP Passthrough — modem only)
  192.168.1.254  ← Still reachable for admin purposes
      │
      ▼
RAXE300 (main router)
  192.168.1.1
      │
      ├── ASUS ROG Desktop (wired)
      ├── MacBook (WiFi)
      └── All other devices
```

---

## Key Learnings

- IP Passthrough alone could not resolve the speed issue while the BGW320 had an internal modem fault — hardware problems must be resolved before software configuration can be effective
- Always verify the modem/ISP hardware is functioning correctly before spending time tuning router settings
- IP Passthrough is the correct long-term configuration for any router placed behind an AT&T BGW320 — it eliminates Double NAT and gives full routing control to your own hardware

---

**Next Step:** [03 — UCG-Fiber & U7 Pro Setup](03-ucg-fiber-u7pro-setup.md)

