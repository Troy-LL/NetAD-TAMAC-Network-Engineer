# Phase 2 — Inter-VLAN Communication

**Prerequisites:** [Phase 1](01-connectivity-testing.md) complete.  
**Primary test host:** **PC-EXEC** (VLAN 10)  
**Screenshot folder:** `screenshots/02-inter-vlan/`

Inter-VLAN routing happens on **CORE-SW** (SVIs). Guest VLAN 90 is routed at the **Edge Router** — guest tests are in [Phase 6](06-guest-network-testing.md).

---

## What this phase proves

Department VLANs can reach other department gateways and hosts. The core switch is performing inter-VLAN routing between SVIs.

---

## Step 1 — Executive → Finance gateway

**Where:** **PC-EXEC** → **Desktop** → **Command Prompt**

```
ping 192.168.20.1
```

**Expected:** Replies from CORE-SW Vlan20 SVI (Finance gateway).

**Screenshot:** `01-exec-ping-finance-gw.png`

---

## Step 2 — Executive → Finance host

**Where:** Same **PC-EXEC Command Prompt**

First confirm **PC-FIN** has an IP (on **PC-FIN**: `ipconfig`). Then from **PC-EXEC**:

```
ping 192.168.20.x
```

Replace `x` with PC-FIN’s host octet (e.g. `ping 192.168.20.10`).

**Expected:** Replies from a host on VLAN 20.

**Screenshot:** `02-exec-ping-finance-pc.png` — show both PCs’ subnets in frame if possible.

---

## Step 3 — Executive → IT gateway

**Where:** Same **PC-EXEC Command Prompt**

```
ping 192.168.50.1
```

**Expected:** Replies from CORE-SW Vlan50 SVI.

**Screenshot:** `03-exec-ping-it-gw.png`

---

## Step 4 — Executive → DNS (cross-VLAN to server VLAN)

**Where:** Same **PC-EXEC Command Prompt**

```
ping 192.168.100.2
```

**Expected:** OK (same as phase 1 — confirms routing to VLAN 100, not just same-subnet).

**Screenshot:** `04-exec-ping-dns-intervlan.png` — optional if you already captured in phase 1; include if your rubric wants inter-VLAN called out separately.

---

## Step 5 — Reverse direction (IT → Executive)

**Where:** **PC-IT** → **Desktop** → **Command Prompt**

```
ping 192.168.10.1
ping 192.168.10.x
```

Use PC-EXEC’s actual host address for the second ping.

**Expected:** Both succeed.

**Screenshot:** `05-it-ping-exec.png`

---

## Step 6 — Verify SVIs on Core (proof for instructor)

**Where:** **CORE-SW** → **CLI** tab

```
enable
show ip interface brief
```

**Expected:** Vlan10, Vlan20, Vlan30, Vlan40, Vlan50, Vlan70, Vlan80, Vlan100, Vlan200 all **up/up** with correct `.1` gateways.

**Screenshot:** `06-core-svi-brief.png`

---

## Phase 2 gate

| # | Test | Pass criteria |
|---|---|---|
| 1 | VLAN 10 → VLAN 20 | Gateway + host ping OK |
| 2 | VLAN 10 → VLAN 50 | Gateway ping OK |
| 3 | VLAN 50 → VLAN 10 | Reverse ping OK |
| 4 | Core SVIs | `show ip interface brief` shows dept SVIs up |
| 5 | Screenshots | Minimum 4 files in `screenshots/02-inter-vlan/` |

**Next:** [Phase 3 — Routing Verification](03-routing-verification.md)
