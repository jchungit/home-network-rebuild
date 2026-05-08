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

## Phase 1 — Double NAT Investigation

### Root Cause Analysis

Running `ipconfig` on the Windows desktop revealed the first issue:

```
Default Gateway: 192.168.1.254  ← AT&T BGW320 gateway IP
```

The network topology at the time:

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

**The problem:** Both the BGW320 and the RAXE300 were acting as routers simultaneously — known as **Double NAT**. Traffic was being translated twice before reaching the internet, creating latency overhead and a throughput bottleneck.

IP Passthrough was configured on the BGW320 to eliminate the Double NAT. Speeds improved temporarily but the problem was not fully resolved.

### Why the BGW320 Causes This

The AT&T BGW320-500 is a **combo device** — it functions as:
1. An **ONT (Optical Network Terminal)** — converts fiber light signals to ethernet
2. A **router** — handles NAT and DHCP by default

AT&T requires the BGW320 to stay connected (it handles authentication with AT&T's fiber network), but it doesn't have to act as the primary router.

---

## Phase 2 — Persistent Speed Issues & AT&T Technician Dispatch

### Symptom: Speed Throttling After Reset

Even after configuring IP Passthrough and ruling out the RAXE300 as a cause, a consistent pattern emerged:

- **After a factory reset of the BGW320** → speeds would briefly return to ~300 Mbps
- **Within a short period** → speeds would throttle back down to ~150 Mbps
- This behavior occurred **regardless of whether the RAXE300 was connected or not**
- Direct wired connection straight from the BGW320 confirmed the RAXE300 was not the cause

This ruled out any software or configuration issue — the problem was isolated to the BGW320 itself.

### AT&T Technician Visit

An AT&T technician was dispatched to diagnose the issue on-site. Findings:

- The existing BGW320-500 unit was diagnosed with an **internal modem fault**
- The internal modem hardware was failing and could not sustain full throughput output regardless of any configuration changes
- It could temporarily reach 300 Mbps after a reset due to the modem briefly reinitializing, but the internal fault caused it to throttle back down to ~150 Mbps shortly after
- No amount of software configuration or router changes could resolve a hardware-level internal modem failure
- The technician replaced the BGW320-500 with a **new unit**

### Result After Modem Replacement

| Device | Faulty Modem (throttled) | New Modem |
|--------|--------------------------|-----------|
| Wired Desktop | ~142–150 Mbps | ~285–295 Mbps stable |
| MacBook WiFi | ~179 Mbps | ~260 Mbps stable |
| iPhone WiFi | ~181 Mbps | ~255 Mbps stable |

Speeds stabilized at near-plan rates and no longer throttled. ✅

---

## Full Diagnostic Timeline

1. **Observed slow speeds** across all devices on 300 Mbps plan
2. **Ran `ipconfig`** → identified Double NAT (BGW320 + RAXE300 both routing)
3. **Configured IP Passthrough** → partial improvement but speeds still inconsistent
4. **Isolated the RAXE300** → connected directly to BGW320, same throttling behavior
5. **Factory reset BGW320** → speeds briefly hit 300 Mbps, then dropped to ~150 Mbps
6. **Contacted AT&T support** → technician dispatched
7. **Technician diagnosed internal modem fault** → hardware unable to sustain output
8. **AT&T replaced the BGW320 unit** → speeds stabilized ✅

---

## Key Learnings

- Double NAT was a contributing factor but **not the root cause** of the speed issue
- A factory reset temporarily masking a hardware problem is a classic sign of **hardware degradation**, not a configuration issue
- An **internal modem fault** can present as an intermittent speed issue — the device appears functional but cannot sustain throughput
- Isolating variables (removing the RAXE300, testing direct wired connections) is essential to pinpoint whether the issue is the ISP hardware or your own equipment
- When self-troubleshooting reaches its limit, escalating to the ISP with documented evidence leads to faster resolution

---

**Next Step:** [02 — IP Passthrough Setup](02-ip-passthrough-setup.md)
