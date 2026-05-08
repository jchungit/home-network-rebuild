# 01 — BGW320-500 Diagnosis: Identifying the Double NAT Problem & Faulty Modem

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
- Port forwarding complexity
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
