# TAMAC Inc. — Packet Tracer Implementation Progress

> **Source of truth:** [`cisco.mdc`](../cisco.mdc) (index) · **Design details:** [`docs/design/`](design/) · **Session logs:** [`docs/logs/`](logs/)

**Last updated:** 2025-06-25  
**Overall progress:** ~100% (security hardening complete; optional demos remain)

---

## Progress at a Glance

| Phase | Status | Notes |
|---|---|---|
| Logical topology placement | ✅ Done | ISP → Router → Core → 4 IDFs + DNS-SVR |
| Physical cabling | ✅ Done | Serial WAN, router/core, DNS, 8 uplink cables |
| Core Switch (3560) | ✅ Done | VLANs, SVIs, DHCP, ACLs, EtherChannel, STP root |
| Edge Router (2911) LAN | ✅ Done | `Gi0/0` = `192.168.200.14/28` (see deviation below) |
| Edge Router WAN / NAT | ✅ Done | ISP-Router (replaces Cloud); Serial up/up; ping 10.0.0.1 5/5 |
| DNS-SVR | ✅ Done | A records + DNS service ON |
| IDF 1-A | ✅ Done | Po1(SU), access ports configured |
| IDF 1-B | ✅ Done | Po2(SU), access ports configured |
| IDF 2-A | ✅ Done | Po3(SU); Fa0/9–14 unused (no Procurement) |
| IDF 2-B | ✅ Done | Po4(SU), access ports configured |
| Access Points | ✅ Done | Guest + 7 corporate APs (EXEC, FIN, CS, OPS, IT, HR, MARK) |
| PCs / Laptops | ✅ Done | ~3 PCs/dept + STAFF-FIN, STAFF-CS, STAFF-IT + Guest laptop |
| Corporate Wi‑Fi | ✅ Done | Per-dept SSIDs `TAMAC-Corp-<dept>` on each VLAN — verified STAFF-FIN |
| SSH (6 devices) | ✅ Done | IDF-2B uses `Tamac2024`; see [08-security-hardening.md](logs/08-security-hardening.md) |
| Port security (4 IDFs) | ✅ Done | Access ports only; uplinks excluded |
| DHCP snooping | ✅ Done (config) | Configured on core + IDFs; **disabled at runtime in PT** for DHCP — [log 09](logs/09-dhcp-snooping-pt-workaround.md) |
| SERVER_ACCESS ACL | ✅ Done (functional) | ACL defined; PT may show Vlan100 inbound "not set" — tests pass |
| End-to-end testing | ✅ Done | Guest isolation + staff Wi‑Fi + wired PCs verified |

**Legend:** ✅ Done · 🟡 Partial · ⬜ Not started

---

## Project Scope — No Procurement

**TAMAC has no Procurement department.** The eight departments are Executive, IT, Operations, Finance, Customer Service, Marketing, HR, and Guest. Never configure VLAN 60, subnet `192.168.60.0/26`, `PROCURE_POOL`, or Procurement ports. IDF 2-A `Fa0/9–14` stay shutdown on VLAN 999.

---

## Implemented Deviations from Design Doc

These differ from `cisco.mdc` on purpose based on Packet Tracer limits and project decisions:

| Topic | Design doc | Actual implementation |
|---|---|---|
| Core Gigabit ports | `Gi0/1`–`Gi0/9` | 3560-24PS only has **2** Gi ports; uplinks use **Fa0/2–7** + **Gi0/1–2** |
| Router → Core port | Core `Gi0/9` | Core **`Fa0/8`** (access VLAN 200) |
| Router LAN IP | `192.168.200.254/28` | **`192.168.200.14/28`** (`.254` invalid on `/28`) |
| Core default route | → `192.168.200.254` | → **`192.168.200.14`** |
| VLAN 60 / Procurement | — | **Not used — not a department** |
| IDF 2-A Fa0/9–14 | — | **Shutdown / VLAN 999** (unused/reserved) |
| GUEST_RESTRICT ACL | On Core Vlan90 (design) | **Edge Router Gi0/0.90** — PT 3560 won't bind ACL to SVI |
| Router → Core port | Core Fa0/8 access VLAN 200 | **Trunk** vlan 90,200 + Gi0/0.90 / Gi0/0.200 subifs |
| SERVER_ACCESS ACL | Original deny blocked core↔DNS | Same-subnet permit first |
| Router WAN module | — | **HWIC-2T** added on 2911 for Serial0/0/0 |
| DHCP snooping runtime | Configured + trusted uplinks | **`no ip dhcp snooping`** on core + IDFs for PT DHCP — see log 09 |

---

## Verified Test Results

| Test | From | To | Result |
|---|---|---|---|
| Core ↔ Router | CORE-SW | `192.168.200.14` | ✅ 80% (4/5) — ARP on first ping |
| Router ↔ Core | Router | `192.168.200.1` | ✅ 100% (5/5) |
| Core ↔ DNS (ACL off) | CORE-SW | `192.168.100.2` | ✅ 100% (5/5) |
| Core ↔ DNS (ACL on) | CORE-SW | `192.168.100.2` | ✅ 100% (5/5) after ACL recreate |
| EtherChannel Po1–Po4 | CORE-SW | All IDFs | ✅ All SU, ports (P) |
| Edge Router → ISP | Router | `10.0.0.1` | ✅ 100% (5/5) |
| Default route | Router | `0.0.0.0/0 → 10.0.0.1` | ✅ In routing table |
| PC5 DHCP on IDF 1-A | PC5 | VLAN 10 Executive | ✅ Working |
| Guest laptop DHCP | Guest laptop | VLAN 90 via AP | ✅ Working |
| Guest isolation ACL | Guest laptop | ping 192.168.10.1 blocked | ✅ Via Edge Router Gi0/0.90 ACL |
| Guest internet | Guest laptop | ping 10.0.0.1 | ✅ Working |
| Staff FIN Wi‑Fi DHCP | STAFF-FIN | VLAN 20 via AP-FIN | ✅ Working |
| Staff FIN → DNS | STAFF-FIN | ping 192.168.100.2 | ✅ 4/4 |
| Staff FIN → inter-VLAN | STAFF-FIN | ping 192.168.10.1 | ✅ 4/4 |
| Staff FIN → internet | STAFF-FIN | ping 10.0.0.1 | ✅ 4/4 |
| Logical end-device topology | All IDFs | PCs + APs + laptops | ✅ Built |

---

## Session Logs

| # | Date | File | Summary |
|---|---|---|---|
| 01 | 2025-06-24 | [01-topology-core-idf-dns.md](logs/01-topology-core-idf-dns.md) | Topology, cabling, core config, router LAN, DNS, all 4 IDFs |
| 02 | 2025-06-24 | [02-wan-isp-router.md](logs/02-wan-isp-router.md) | ISP-Router, default route, WAN ping verified |
| 03 | 2025-06-24 | [03-pc5-idf-dhcp.md](logs/03-pc5-idf-dhcp.md) | PC5 on IDF 1-A, DHCP after reset recovery |
| 04 | 2025-06-24 | [04-guest-ap-dhcp.md](logs/04-guest-ap-dhcp.md) | Guest AP, ACL fix, DHCP + isolation verified |
| 05 | 2025-06-24 | [05-end-devices-staff-wifi.md](logs/05-end-devices-staff-wifi.md) | +2 PCs/dept port map, corporate APs, staff laptops, full CLI |
| 06 | 2025-06-24 | [06-guest-acl-fix.md](logs/06-guest-acl-fix.md) | Guest ping 192.168.10.1 should fail — ACL/SSID troubleshooting |
| 07 | 2025-06-24 | [07-guest-acl-router-fix.md](logs/07-guest-acl-router-fix.md) | Guest ACL on Edge Router — isolation verified |
| 08 | 2025-06-24 | [08-security-hardening.md](logs/08-security-hardening.md) | SSH, port security, DHCP snooping, SERVER_ACCESS |
| 09 | 2025-06-25 | [09-dhcp-snooping-pt-workaround.md](logs/09-dhcp-snooping-pt-workaround.md) | DHCP snooping disabled for PT; deliverables screenshots |

---

## Capstone deliverables

All submission artifacts: [deliverables/README.md](deliverables/README.md)

| # | Deliverable | Path |
|---|---|---|
| 1 | Network diagram | [deliverables/01-network-diagram/](deliverables/01-network-diagram/) |
| 2 | Device configs | [deliverables/02-device-configs/](deliverables/02-device-configs/) |
| 3 | Screenshots | [deliverables/03-screenshots/](deliverables/03-screenshots/) |
| 4 | IP addressing | [deliverables/04-ip-addressing.md](deliverables/04-ip-addressing.md) |
| 5 | VLAN tables | [deliverables/05-vlan-table.md](deliverables/05-vlan-table.md) |

---

## Next Steps

1. **File → Save** your `.pkt` after any lab changes
2. Use [deliverables/03-screenshots/](deliverables/03-screenshots/) for presentation proof
3. Share [GROUP-REFERENCE.md](GROUP-REFERENCE.md) with groupmates for full context
4. Optional demos: port-security violation, EtherChannel failover

---

*This file is the master progress index. Detailed commands and CLI output live in `docs/logs/`.*
