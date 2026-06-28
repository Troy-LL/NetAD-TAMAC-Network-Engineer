# Phase 4 — DHCP Verification

**Prerequisites:** [Phase 3](03-routing-verification.md) complete.  
**Devices:** **CORE-SW** (staff pools), **Edge Router** (guest pool), **PC-EXEC**, **STAFF-FIN**, **Guest laptop**  
**Screenshot folder:** `screenshots/04-dhcp/`

---

## What this phase proves

DHCP scopes are defined on the correct devices, clients receive correct gateway/DNS options, and leases appear in server binding tables.

**PT note:** If DHCP fails, disable snooping on core + IDFs first — [log 09](../../logs/09-dhcp-snooping-pt-workaround.md).

---

## Step 1 — DHCP pools on Core

**Where:** **CORE-SW** → **CLI** tab

```
enable
show ip dhcp pool
show ip dhcp binding
```

**Expected pools (names may vary slightly):** `EXEC_POOL`, `FIN_POOL`, `OPS_POOL`, `CS_POOL`, `IT_POOL`, `HR_POOL`, `MARK_POOL` — each with correct network, default-router `.1`, DNS `192.168.100.2`.

**Screenshot:** `01-core-dhcp-pools.png`  
**Screenshot:** `02-core-dhcp-bindings.png` — active leases after clients renew.

---

## Step 2 — Wired staff DHCP (Executive)

**Where:** **PC-EXEC** → **Desktop** → **Command Prompt**

```
ipconfig /release
ipconfig /renew
ipconfig
```

**Expected:** `192.168.10.x`, gateway `192.168.10.1`, DNS `192.168.100.2`.

**Screenshot:** `03-exec-dhcp-renew.png`

---

## Step 3 — Wired staff DHCP (second VLAN — Finance)

**Where:** **PC-FIN** → **Desktop** → **Command Prompt**

```
ipconfig /renew
ipconfig
```

**Expected:** `192.168.20.x`, gateway `192.168.20.1`, DNS `192.168.100.2`.

**Screenshot:** `04-fin-dhcp.png`

---

## Step 4 — Corporate Wi‑Fi DHCP

**Where:** **STAFF-FIN** laptop → **Desktop** → **PC Wireless**

1. Connect to SSID **`TAMAC-Corp-fin`** (passphrase `TAMACstaff2024!`).
2. Open **Command Prompt**:

```
ipconfig /renew
ipconfig
```

**Expected:** `192.168.20.x` on wireless — same VLAN as Finance wired PCs.

**Screenshot:** `05-staff-fin-wifi-dhcp.png` — wireless tab + ipconfig.

---

## Step 5 — Guest DHCP on Edge Router

**Where:** **Edge Router** → **CLI** tab

```
show ip dhcp pool
show ip dhcp binding
```

**Expected:** Guest pool on `Gi0/0.90` network `192.168.90.0/25`, default-router `192.168.90.1`, DNS `8.8.8.8`.

**Screenshot:** `06-edge-router-guest-pool.png`

---

## Step 6 — Guest laptop lease

**Where:** **Guest laptop** → **Desktop** → **PC Wireless**

1. Connect to **`TAMAC-Guest`** (passphrase `Guest123!`).
2. **Command Prompt**:

```
ipconfig /renew
ipconfig
```

**Expected:**

| Field | Value |
|---|---|
| IP | `192.168.90.x` |
| Gateway | `192.168.90.1` |
| DNS | `8.8.8.8` |

**Screenshot:** `07-guest-laptop-dhcp.png`

---

## Step 7 — Cross-check lease on router

**Where:** **Edge Router** → **CLI** tab

```
show ip dhcp binding
```

**Expected:** Binding for guest laptop MAC with `192.168.90.x`.

**Screenshot:** `08-edge-router-guest-binding.png`

---

## Phase 4 gate

| # | Test | Pass criteria |
|---|---|---|
| 1 | Core pools | All dept pools present with correct options |
| 2 | Wired DHCP | EXEC + FIN get correct VLAN addresses |
| 3 | Wi‑Fi DHCP | STAFF-FIN gets Finance VLAN address |
| 4 | Guest DHCP | `192.168.90.x` from router pool |
| 5 | Screenshots | Minimum 6 files in `screenshots/04-dhcp/` |

**Next:** [Phase 5 — Security Testing](05-security-testing.md)
