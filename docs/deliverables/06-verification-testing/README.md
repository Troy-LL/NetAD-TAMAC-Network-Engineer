# TAMAC Inc. — Sequential Verification Testing

> **Packet Tracer file:** `Final Proj.pkt` (repo root)  
> **Rule:** Complete each phase in order. Do not start the next phase until every screenshot for the current phase is captured and every test passes.

Save screenshots into the matching subfolder under [`screenshots/`](screenshots/). Use the filenames listed in each phase so your submission stays organized.

**Credentials (lab):** `admin` / `TamacAdmin2024!` · IDF 2-B uses `Tamac2024` · domain `tamac.local`

---

## Phase overview

| Phase | Topic | Screenshot folder | Gate (must pass before next) |
|---|---|---|---|
| **1** | [Connectivity Testing](01-connectivity-testing.md) | `screenshots/01-connectivity/` | Staff PC gets IP, pings gateway, DNS, internet |
| **2** | [Inter-VLAN Communication](02-inter-vlan-communication.md) | `screenshots/02-inter-vlan/` | At least two dept VLANs reach each other |
| **3** | [Routing Verification](03-routing-verification.md) | `screenshots/03-routing/` | Core + Edge Router show correct routes |
| **4** | [DHCP Verification](04-dhcp-verification.md) | `screenshots/04-dhcp/` | Wired + wireless clients get correct leases |
| **5** | [Security Testing](05-security-testing.md) | `screenshots/05-security/` | All seven security sub-tests documented |
| **6** | [Guest Network Testing](06-guest-network-testing.md) | `screenshots/06-guest/` | Guest DHCP, isolation, internet only |

---

## Devices used across all phases

| Device | VLAN | Role in testing |
|---|---|---|
| **PC-EXEC** | 10 (Executive) | Primary wired staff PC — phases 1–4 |
| **PC-IT** | 50 (IT) | SSH, security CLI, routing — phases 3–5 |
| **STAFF-FIN** (laptop) | 20 (Finance) | Optional wireless DHCP proof — phase 4 |
| **Guest laptop** | 90 (Guest) | Phases 5 (guest ACL) and 6 |
| **CORE-SW** | 200 (mgmt) | DHCP pools, ACLs, routing, snooping |
| **Edge Router** | 200 + 90 | Default route, NAT, GUEST_RESTRICT ACL |
| **IDF 1-A** | 200 (mgmt) | Port security demo |

---

## Before you start

1. Open `Final Proj.pkt` in Packet Tracer.
2. Switch to **Simulation** mode off (Realtime).
3. If DHCP fails on any PC, see [log 09](../../logs/09-dhcp-snooping-pt-workaround.md) — run `no ip dhcp snooping` on **CORE-SW** and all **IDFs** before phase 1.
4. Confirm simulation links are green (no err-disabled ports). If an AP port is red, see [log 10](../../logs/10-ap-port-security-multi-client.md).

---

## Quick reference — where commands go

| Location in Packet Tracer | How to open | Used for |
|---|---|---|
| **PC Command Prompt** | Click PC → **Desktop** tab → **Command Prompt** | `ipconfig`, `ping`, `ssh`, `tracert` |
| **Switch/Router CLI** | Click device → **CLI** tab → press Enter if needed → `enable` | `show`, `ping`, `configure terminal` |
| **Wireless client** | Click laptop → **Desktop** → **PC Wireless** → connect to SSID | Guest / staff Wi‑Fi DHCP |
| **Device Config tab** | Click device → **Config** tab | Optional ACL bind on Core Vlan100 (PT quirk) |

---

## Related deliverables

| Need | Location |
|---|---|
| Existing proof screenshots | [../03-screenshots/](../03-screenshots/) |
| Running configs | [../02-device-configs/](../02-device-configs/) |
| IP / VLAN tables | [../04-ip-addressing.md](../04-ip-addressing.md) · [../05-vlan-table.md](../05-vlan-table.md) |
| Build status | [../../PROGRESS.md](../../PROGRESS.md) |

---

*Built reality in logs wins over design docs. If a test fails, check [GROUP-REFERENCE.md](../../GROUP-REFERENCE.md) §15 before changing config.*
