# Phase 1 — Connectivity Testing

**Prerequisites:** None (start here).  
**Primary test host:** **PC-EXEC** (Executive, VLAN 10, IDF 1-A Fa0/1)  
**Screenshot folder:** `screenshots/01-connectivity/`

Complete every item below before opening [Phase 2](02-inter-vlan-communication.md).

---

## What this phase proves

End devices on the corporate LAN can obtain addressing, reach their default gateway, reach internal DNS, and reach the internet through NAT.

---

## Step 1 — Renew DHCP on PC-EXEC

**Where:** **PC-EXEC** → **Desktop** → **Command Prompt**

```
ipconfig /release
ipconfig /renew
ipconfig
```

**Expected:**

| Field | Value |
|---|---|
| IP Address | `192.168.10.x` (pool range `.10`–`.62`) |
| Subnet Mask | `255.255.255.192` |
| Default Gateway | `192.168.10.1` |
| DNS Server | `192.168.100.2` |

**Screenshot:** `01-exec-pc-dhcp.png` — full `ipconfig` output visible.

---

## Step 2 — Ping default gateway (L3 on Core)

**Where:** Same **PC-EXEC Command Prompt**

```
ping 192.168.10.1
```

**Expected:** Replies from `192.168.10.1` (CORE-SW Vlan10 SVI). 4/4 or 5/5 success.

**Screenshot:** `02-exec-ping-gateway.png`

---

## Step 3 — Ping DNS server

**Where:** Same **PC-EXEC Command Prompt**

```
ping 192.168.100.2
```

**Expected:** Replies from DNS-SVR. Proves access VLAN + server subnet reachability.

**Screenshot:** `03-exec-ping-dns.png`

---

## Step 4 — Ping internet (ISP)

**Where:** Same **PC-EXEC Command Prompt**

```
ping 10.0.0.1
```

**Expected:** Replies from ISP-Router WAN side. Proves default route on Edge Router + NAT.

**Screenshot:** `04-exec-ping-internet.png`

---

## Step 5 — Core ↔ Edge Router link (optional but recommended)

**Where:** **CORE-SW** → **CLI** tab

```
enable
ping 192.168.200.14
```

**Expected:** Replies from Edge Router Gi0/0.200 (`192.168.200.14`).

**Screenshot:** `05-core-ping-edge-router.png`

---

## Phase 1 gate — all must pass

| # | Test | Pass criteria |
|---|---|---|
| 1 | PC-EXEC DHCP | `192.168.10.x`, GW `.1`, DNS `.100.2` |
| 2 | PC-EXEC → gateway | Ping OK |
| 3 | PC-EXEC → DNS | Ping OK |
| 4 | PC-EXEC → internet | Ping `10.0.0.1` OK |
| 5 | Screenshots | All four (or five) files saved in `screenshots/01-connectivity/` |

**Next:** [Phase 2 — Inter-VLAN Communication](02-inter-vlan-communication.md)
