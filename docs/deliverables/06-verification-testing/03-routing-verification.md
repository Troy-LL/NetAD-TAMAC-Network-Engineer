# Phase 3 — Routing Verification

**Prerequisites:** [Phase 2](02-inter-vlan-communication.md) complete.  
**Devices:** **CORE-SW**, **Edge Router**, **PC-IT** (for traceroute)  
**Screenshot folder:** `screenshots/03-routing/`

---

## What this phase proves

Connected routes exist for every VLAN SVI, the core knows how to reach the guest subnet via the router, and the edge router has a default route to the ISP with NAT for internet access.

---

## Step 1 — Core routing table

**Where:** **CORE-SW** → **CLI** tab

```
enable
show ip route
```

**Expected (highlights):**

| Route | Via / interface |
|---|---|
| `192.168.10.0/26` … `192.168.80.0/26` | Connected — respective `VlanXX` |
| `192.168.100.0/28` | Connected — `Vlan100` |
| `192.168.200.0/28` | Connected — `Vlan200` |
| `192.168.90.0/25` | Static → `192.168.200.14` (guest via Edge Router) |
| `0.0.0.0/0` | Default → `192.168.200.14` |

**Screenshot:** `01-core-show-ip-route.png` — scroll if needed so guest static and default route are visible.

---

## Step 2 — Edge Router routing table

**Where:** **Edge Router** → **CLI** tab

```
enable
show ip route
```

**Expected:**

| Route | Notes |
|---|---|
| `192.168.200.0/28` | Connected — `Gi0/0.200` |
| `192.168.90.0/25` | Connected — `Gi0/0.90` |
| `10.0.0.0/30` | Connected — WAN Serial |
| `0.0.0.0/0` | Default → `10.0.0.1` (ISP) |

**Screenshot:** `02-edge-router-show-ip-route.png`

---

## Step 3 — Core default gateway to router

**Where:** **CORE-SW** → **CLI** tab

```
show ip route 0.0.0.0
ping 192.168.200.14
```

**Expected:** Default route points to `192.168.200.14`; ping succeeds.

**Screenshot:** `03-core-default-route-ping.png`

---

## Step 4 — Edge Router NAT / interfaces

**Where:** **Edge Router** → **CLI** tab

```
show ip interface brief
show ip nat translations
```

**Expected:** `Gi0/0.200` and `Gi0/0.90` up; `Serial0/0/0` up; NAT translations may appear after PCs pinged internet.

**Screenshot:** `04-edge-router-interfaces-nat.png`

---

## Step 5 — Traceroute from staff PC to internet

**Where:** **PC-IT** → **Desktop** → **Command Prompt**

```
tracert 10.0.0.1
```

**Expected path (approximate):** PC → `192.168.50.1` (core) → `192.168.200.14` (router) → `10.0.0.1` (ISP).

**Screenshot:** `05-it-tracert-internet.png`

---

## Step 6 — Guest static route on Core

**Where:** **CORE-SW** → **CLI** tab

```
show ip route 192.168.90.0
```

**Expected:** `192.168.90.0/25` via `192.168.200.14`.

**Screenshot:** `06-core-guest-static-route.png`

---

## Phase 3 gate

| # | Test | Pass criteria |
|---|---|---|
| 1 | Core routes | All dept SVIs + static to 90.x + default |
| 2 | Router routes | Subinterfaces + WAN + default |
| 3 | Traceroute | Shows core then router hops |
| 4 | Screenshots | Minimum 4 files in `screenshots/03-routing/` |

**Next:** [Phase 4 — DHCP Verification](04-dhcp-verification.md)
