# Phase 6 — Guest Network Testing

**Prerequisites:** [Phase 5](05-security-testing.md) complete.  
**Primary test host:** **Guest laptop** (SSID `TAMAC-Guest`)  
**Screenshot folder:** `screenshots/06-guest/`

This phase is the **end-to-end guest user story**: connect, get DHCP, verify isolation from all corporate resources, confirm internet-only access.

---

## What this phase proves

Guests receive addresses from the Edge Router guest pool, cannot reach internal VLANs or DNS, and can reach the internet through NAT.

---

## Step 1 — Connect to guest Wi‑Fi

**Where:** **Guest laptop** → **Desktop** → **PC Wireless**

| Setting | Value |
|---|---|
| SSID | `TAMAC-Guest` |
| Security | WPA2-PSK |
| Passphrase | `Guest123!` |

**Expected:** Link connected to **AP-Guest** (IDF 1-A Fa0/17).

**Screenshot:** `01-guest-wireless-connected.png` — PC Wireless tab showing SSID.

---

## Step 2 — Guest DHCP addressing

**Where:** **Guest laptop** → **Desktop** → **Command Prompt**

```
ipconfig /release
ipconfig /renew
ipconfig
```

**Expected:**

| Field | Value |
|---|---|
| IP Address | `192.168.90.x` |
| Subnet Mask | `255.255.255.128` |
| Default Gateway | `192.168.90.1` |
| DNS | `8.8.8.8` |

If you see `192.168.10.x` or any non-90 address, you are on the **wrong SSID** — disconnect and reconnect to `TAMAC-Guest` only.

**Screenshot:** `02-guest-laptop-dhcp.png`

---

## Step 3 — Isolation from Executive VLAN

**Where:** Same **Guest laptop Command Prompt**

```
ping 192.168.10.1
ping 192.168.10.50
```

Use a real Executive host IP if PC-EXEC shows a different address.

**Expected:** `Destination host unreachable from 192.168.90.1` or request timeout — **not** successful replies.

**Screenshot:** `03-guest-blocked-exec.png`

---

## Step 4 — Isolation from multiple internal subnets

**Where:** Same **Guest laptop Command Prompt**

```
ping 192.168.20.1
ping 192.168.50.1
ping 192.168.100.2
ping 192.168.200.1
```

**Expected:** All fail — Finance, IT, DNS, and management are unreachable.

**Screenshot:** `04-guest-blocked-internal-subnets.png` — one capture with multiple failed pings is fine.

---

## Step 5 — Guest internet access

**Where:** Same **Guest laptop Command Prompt**

```
ping 10.0.0.1
ping 8.8.8.8
```

**Expected:** At least `10.0.0.1` (ISP) succeeds. `8.8.8.8` may succeed if PT models external DNS.

**Screenshot:** `05-guest-ping-internet.png`

---

## Step 6 — Prove gateway is Edge Router (not Core)

**Where:** **Edge Router** → **CLI** tab

```
show ip interface GigabitEthernet0/0.90
show ip dhcp binding
```

**Expected:** `Gi0/0.90` is `192.168.90.1/25`, up/up; guest laptop binding listed.

**Where:** **CORE-SW** → **CLI**

```
show ip interface Vlan90
```

**Expected:** Vlan90 **administratively down** (guest L3 lives on router).

**Screenshot:** `06-guest-gateway-on-router.png` — router subinterface + core Vlan90 down.

---

## Step 7 — Multi-guest on AP (optional)

If your rubric requires multiple concurrent guests:

1. Add a second laptop to `TAMAC-Guest`.
2. Both get `192.168.90.x` addresses.
3. AP port **IDF 1-A Fa0/17** stays up (port-security `maximum 50`).

**Screenshot:** `07-multi-guest-dhcp.png`

---

## Phase 6 gate — final submission checklist

| # | Test | Pass criteria |
|---|---|---|
| 1 | Wi‑Fi | Connected to `TAMAC-Guest` only |
| 2 | DHCP | `192.168.90.x`, GW `.90.1`, DNS `8.8.8.8` |
| 3 | Isolation | All internal pings fail |
| 4 | Internet | Ping `10.0.0.1` OK |
| 5 | Architecture | Guest GW on Edge Router Gi0/0.90 |
| 6 | Screenshots | Minimum 5 files in `screenshots/06-guest/` |

---

## All phases complete

When phases 1–6 pass:

1. **File → Save** `Final Proj.pkt`
2. Copy all screenshots into [`screenshots/`](screenshots/) subfolders
3. Reference this package in your presentation: [README.md](README.md)
4. Existing capstone artifacts remain in [../03-screenshots/](../03-screenshots/) — you may reuse images if filenames match

**Related logs:** [04-guest-ap-dhcp.md](../../logs/04-guest-ap-dhcp.md) · [07-guest-acl-router-fix.md](../../logs/07-guest-acl-router-fix.md)
