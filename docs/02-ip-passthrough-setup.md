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
