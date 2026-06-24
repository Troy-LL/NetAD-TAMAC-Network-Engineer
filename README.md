# TAMAC Inc. — Enterprise Network (Packet Tracer Capstone)

A secure, highly available enterprise LAN for **TAMAC Inc.**, a fictional e-commerce aggregator with 200–250 employees across a two-story HQ. Built in **Cisco Packet Tracer** as a networking capstone: inter-VLAN routing, DHCP, DNS, Wi‑Fi, NAT, ACLs, EtherChannel, and device hardening.

**Status:** ~100% complete — see [docs/PROGRESS.md](docs/PROGRESS.md) for verified tests and session history.

---

## Quick start

| If you want to… | Start here |
|---|---|
| Understand the whole project in one sitting | This README, then [docs/GROUP-REFERENCE.md](docs/GROUP-REFERENCE.md) |
| See what's done and what differs from design | [docs/PROGRESS.md](docs/PROGRESS.md) |
| Find commands and CLI output from a build session | [docs/logs/](docs/logs/) (read `01` through `08` in order) |
| Look up VLANs, ports, ACLs, or Wi‑Fi details | [docs/design/](docs/design/) (one topic file at a time) |
| Submit or review capstone deliverables | [docs/deliverables/](docs/deliverables/) |
| Open the live network | `Final Proj.pkt` in the repo root (Packet Tracer) |

**Hard rule:** TAMAC has **no Procurement department**. Never use VLAN 60, `192.168.60.0/26`, or Procurement ports. If design docs conflict with [PROGRESS.md](docs/PROGRESS.md) or [logs/](docs/logs/), **trust what was actually built and logged**.

---

## Repository layout

```
NetAD/
├── README.md                 ← You are here (project overview)
├── cisco.mdc                 ← Agent/source-of-truth index + quick reference
├── .cursor/rules/            ← Cursor rules for AI-assisted work on this repo
└── docs/
    ├── PROGRESS.md           ← Master status, deviations, test results
    ├── GROUP-REFERENCE.md    ← Full teammate onboarding (credentials, quirks, defense)
    ├── design/               ← Topic deep-dives (VLANs, routing, security, ports, etc.)
    ├── logs/                 ← Session logs with real commands + verified output
    └── deliverables/         ← Capstone submission package (diagrams, configs, screenshots, tables)
```

### What each layer is for

| Path | Purpose |
|---|---|
| `cisco.mdc` | Short index: topology diagram, VLAN table, key IPs, port map. Read first for alignment. |
| `docs/PROGRESS.md` | Living checklist — phases complete, intentional deviations from design, ping test matrix. |
| `docs/GROUP-REFERENCE.md` | Longer onboarding doc: credentials, Packet Tracer quirks, troubleshooting, defense checklist. |
| `docs/design/*.md` | Original design broken into topics. Use when you need detail on one area. |
| `docs/logs/NN-*.md` | Chronological build journal. **Built reality wins** over design when they disagree. |
| `docs/deliverables/` | Five capstone deliverables — start at [deliverables/README.md](docs/deliverables/README.md). |

---

## Capstone deliverables

All submission artifacts live under [docs/deliverables/](docs/deliverables/):

| # | Deliverable | Location |
|---|---|---|
| 1 | Network diagram | [01-network-diagram/](docs/deliverables/01-network-diagram/) |
| 2 | Device configurations | [02-device-configs/](docs/deliverables/02-device-configs/) |
| 3 | Screenshots (proof tests) | [03-screenshots/](docs/deliverables/03-screenshots/) |
| 4 | IP addressing tables | [04-ip-addressing.md](docs/deliverables/04-ip-addressing.md) |
| 5 | VLAN tables | [05-vlan-table.md](docs/deliverables/05-vlan-table.md) |

---

## How the network works

### Big picture

Traffic flows through a classic three-tier layout: **WAN edge → core (L3) → access switches (L2) → end devices**.

```
[ ISP-Router 2911 ] ----Serial 10.0.0.0/30---- [ Edge Router 2911 ]
                                                      |
                                              Gi0/0 (trunk: VLAN 90, 200)
                                                      |
                                               [ CORE-SW 3560 ]
                                                /  |  |  |  \
                                          Fa0/1  Po1 Po2 Po3 Po4
                                            |     |   |   |   |
                                        DNS-SVR IDF  IDF IDF IDF
                                                1-A 1-B 2-A 2-B
```

- **ISP-Router** simulates the internet (Serial link with clock rate — Cloud-PT was replaced because it lacks clock rate in Packet Tracer).
- **Edge Router (2911)** handles WAN NAT/PAT, default routing to the ISP, guest gateway + guest ACL, and a management subinterface.
- **CORE-SW (3560)** is the L3 hub: SVIs per VLAN, DHCP for staff VLANs, inter-VLAN routing, EtherChannel uplinks, STP root, and server ACL.
- **Four IDF switches (2960)** on Floors 1 and 2 provide access ports for PCs, APs, and laptops. Each IDF has a dual-link EtherChannel uplink to the core.
- **DNS-SVR** sits on VLAN 100 in the MDF (IT server room on Floor 2).

### Physical placement

| Location | Devices |
|---|---|
| **MDF (Floor 2 — IT server room)** | Edge Router, CORE-SW, DNS-SVR |
| **IDF 1-A (Floor 1)** | Executive, Finance, Operations, Guest lobby AP |
| **IDF 1-B (Floor 1)** | Customer Service, additional Floor 1 users |
| **IDF 2-A (Floor 2)** | IT department |
| **IDF 2-B (Floor 2)** | HR, Marketing |

### Departments and VLANs

Eight departments — **no Procurement**:

| Department | VLAN | Subnet | Gateway |
|---|---|---|---|
| Executive | 10 | 192.168.10.0/26 | 192.168.10.1 |
| Finance | 20 | 192.168.20.0/26 | 192.168.20.1 |
| Operations | 30 | 192.168.30.0/25 | 192.168.30.1 |
| Customer Service | 40 | 192.168.40.0/25 | 192.168.40.1 |
| IT | 50 | 192.168.50.0/26 | 192.168.50.1 |
| HR | 70 | 192.168.70.0/26 | 192.168.70.1 |
| Marketing | 80 | 192.168.80.0/26 | 192.168.80.1 |
| Guest | 90 | 192.168.90.0/25 | **192.168.90.1** (Edge Router) |
| Servers (DNS) | 100 | 192.168.100.0/28 | 192.168.100.1 |
| Management | 200 | 192.168.200.0/28 | 192.168.200.1 |

VLAN 999 is used for unused/shutdown ports (e.g. IDF 2-A Fa0/9–14).

### Key device IPs

| Device | IP | Role |
|---|---|---|
| CORE-SW (Vlan200 SVI) | 192.168.200.1/28 | STP root, inter-VLAN routing, staff DHCP |
| Edge Router Gi0/0.200 | 192.168.200.14/28 | Management + default route target for core |
| Edge Router Gi0/0.90 | 192.168.90.1/25 | Guest gateway, guest DHCP, GUEST_RESTRICT ACL |
| Edge Router WAN | 10.0.0.2/30 | NAT to ISP at 10.0.0.1 |
| DNS-SVR | 192.168.100.2/28 | Internal DNS (all staff DHCP pools point here) |
| IDF 1-A – 2-B mgmt | 192.168.200.2–5/28 | Switch management on VLAN 200 |

> The edge router uses **192.168.200.14**, not `.254`. A `/28` subnet only allows host addresses `.1`–`.14`.

### Core switch port map

The 3560-24PS has only **two** Gigabit ports (`Gi0/1–2`). Other uplinks use FastEthernet.

| Core port | Connects to |
|---|---|
| Fa0/1 | DNS-SVR (access VLAN 100) |
| Fa0/2–3 | IDF 1-B (EtherChannel Po2) |
| Fa0/4–5 | IDF 2-A (EtherChannel Po3) |
| Fa0/6–7 | IDF 2-B (EtherChannel Po4) |
| Fa0/8 | Edge Router Gi0/0 (trunk VLANs 90, 200) |
| Gi0/1–2 | IDF 1-A (EtherChannel Po1) |

### How traffic moves

**Staff PC or laptop (wired or per-dept `TAMAC-Corp-<dept>` Wi‑Fi)**

1. Device gets an IP via DHCP from the core (gateway = department SVI, DNS = `192.168.100.2`).
2. Same-VLAN traffic stays on the local IDF switch.
3. Cross-VLAN traffic goes up the EtherChannel trunk to CORE-SW, which routes between SVIs.
4. Internet-bound traffic hits the core default route → Edge Router (`192.168.200.14`) → NAT → ISP (`10.0.0.1`).

**Guest laptop (`TAMAC-Guest` SSID, VLAN 90)**

1. DHCP from the **Edge Router** subinterface Gi0/0.90 (not the core).
2. Default gateway is `192.168.90.1` on the router.
3. **GUEST_RESTRICT** ACL on Gi0/0.90 blocks all internal subnets; internet via NAT still works.
4. Guest must show a `192.168.90.x` address. If it gets a staff subnet, it's on the wrong SSID.

**DNS lookups**

Staff devices use `192.168.100.2`. The SERVER_ACCESS ACL on the core permits department VLANs to reach the server subnet while blocking unauthorized access.

### High availability

Each IDF connects to the core with **two cables** in an **LACP EtherChannel** (Po1–Po4). **Rapid PVST+** runs with the core as root bridge. Unplugging one uplink link leaves the port-channel up — traffic continues on the remaining member port.

---

## Security model

| Feature | Where | What it does |
|---|---|---|
| **GUEST_RESTRICT ACL** | Edge Router Gi0/0.90 | Blocks guest from internal VLANs; allows internet |
| **SERVER_ACCESS ACL** | CORE-SW (Vlan100) | Permits dept VLANs → DNS; denies others |
| **SSH v2** | Core, router, all 4 IDFs | Encrypted management; telnet disabled |
| **Port security** | IDF access ports | Max 1 MAC, sticky learning, violation shutdown |
| **DHCP snooping** | Core + all IDFs (config) | Designed on uplinks; may be **disabled in PT** for DHCP to work — see [log 09](docs/logs/09-dhcp-snooping-pt-workaround.md) |
| **EtherChannel + RSTP** | Core ↔ each IDF | Redundant uplinks with loop prevention |

Lab credentials (Packet Tracer only): `admin` / `TamacAdmin2024!` — see [GROUP-REFERENCE.md](docs/GROUP-REFERENCE.md) for SSH targets and the IDF 2-B password exception.

Full ACL definitions, port ranges, and hardening steps: [docs/design/security.md](docs/design/security.md) and [docs/logs/08-security-hardening.md](docs/logs/08-security-hardening.md).

---

## Wi‑Fi

| SSID | VLAN | Users |
|---|---|---|
| `TAMAC-Corp-exec` | 10 | Executive staff laptops — WPA2-PSK |
| `TAMAC-Corp-fin` | 20 | Finance staff laptops |
| `TAMAC-Corp-ops` | 30 | Operations staff laptops |
| `TAMAC-Corp-cs` | 40 | Customer Service staff laptops |
| `TAMAC-Corp-it` | 50 | IT staff laptops |
| `TAMAC-Corp-hr` | 70 | HR staff laptops |
| `TAMAC-Corp-mark` | 80 | Marketing staff laptops |
| `TAMAC-Guest` | 90 | Guest laptop — lobby AP on IDF 1-A |

Seven corporate APs (one per department, each with a **department-specific SSID** so devices auto-join the correct VLAN) plus one guest AP. All corporate SSIDs share the passphrase `TAMACstaff2024!`. Port assignments: [docs/design/end-devices.md](docs/design/end-devices.md) and [docs/design/corporate-wifi.md](docs/design/corporate-wifi.md).

---

## Important deviations from original design

These are intentional — based on Packet Tracer limits and verified during the build:

| Topic | Design doc said | What we built |
|---|---|---|
| Core Gigabit ports | Gi0/1–9 | Only 2 Gi ports exist; uplinks use Fa0/2–7 + Gi0/1–2 |
| Router LAN IP | 192.168.200.254/28 | **192.168.200.14/28** (.254 invalid on /28) |
| Router ↔ Core link | Access VLAN 200 | **Trunk** VLANs 90 + 200 with router subinterfaces |
| Guest gateway | Core Vlan90 SVI | **Edge Router Gi0/0.90** (core Vlan90 shutdown) |
| Guest ACL | Core Vlan90 inbound | **Edge Router Gi0/0.90** (PT won't bind ACL on 3560 SVI) |
| ISP simulation | Cloud-PT | **ISP-Router (2911)** with HWIC-2T serial |
| DHCP snooping trust | Port-channel interfaces | **Physical uplink ports** only; may need `no ip dhcp snooping` in PT ([log 09](docs/logs/09-dhcp-snooping-pt-workaround.md)) |

Full deviation table: [docs/PROGRESS.md](docs/PROGRESS.md#implemented-deviations-from-design-doc).

---

## Packet Tracer quirks (read before changing anything)

These caused real bugs during the build:

1. **SVI ACL binding fails silently** — `ip access-group` on Vlan90/Vlan100 may accept in CLI but not save. Guest ACL was moved to the edge router ([log 07](docs/logs/07-guest-acl-router-fix.md)).
2. **Port-channel won't take DHCP snooping trust** — trust physical member ports instead.
3. **ACL line order matters** — editing appends lines; delete and recreate ACLs in correct order.
4. **SSH password special characters** — IDF 2-B uses `Tamac2024` instead of `TamacAdmin2024!`.

More troubleshooting: [docs/GROUP-REFERENCE.md § 11 & 15](docs/GROUP-REFERENCE.md).

---

## Design documentation (by topic)

| Topic | File |
|---|---|
| Company, topology, hardware | [docs/design/overview.md](docs/design/overview.md) |
| VLANs and IP addressing | [docs/design/vlans-and-ip.md](docs/design/vlans-and-ip.md) |
| Routing and DHCP pools | [docs/design/routing-dhcp.md](docs/design/routing-dhcp.md) |
| ACLs, SSH, port security | [docs/design/security.md](docs/design/security.md) |
| Guest network | [docs/design/guest-network.md](docs/design/guest-network.md) |
| Corporate Wi‑Fi | [docs/design/corporate-wifi.md](docs/design/corporate-wifi.md) |
| PCs, laptops, AP port map | [docs/design/end-devices.md](docs/design/end-devices.md) |
| EtherChannel and Rapid STP | [docs/design/high-availability.md](docs/design/high-availability.md) |
| DNS server | [docs/design/dns.md](docs/design/dns.md) |
| Cables, ports, IDF layouts | [docs/design/ports-and-cabling.md](docs/design/ports-and-cabling.md) |
| PT tips and defense checklist | [docs/design/packet-tracer-guide.md](docs/design/packet-tracer-guide.md) |

---

## Session logs (build history)

Read in order to follow how the network was built:

| # | File | Summary |
|---|---|---|
| 01 | [01-topology-core-idf-dns.md](docs/logs/01-topology-core-idf-dns.md) | Topology, core VLANs/SVIs/DHCP, all 4 IDFs, DNS |
| 02 | [02-wan-isp-router.md](docs/logs/02-wan-isp-router.md) | ISP-Router, default route, WAN ping |
| 03 | [03-pc5-idf-dhcp.md](docs/logs/03-pc5-idf-dhcp.md) | End-device DHCP |
| 04 | [04-guest-ap-dhcp.md](docs/logs/04-guest-ap-dhcp.md) | Guest AP and early ACL work |
| 05 | [05-end-devices-staff-wifi.md](docs/logs/05-end-devices-staff-wifi.md) | PCs, APs, corporate Wi‑Fi |
| 06 | [06-guest-acl-fix.md](docs/logs/06-guest-acl-fix.md) | Guest isolation troubleshooting on core |
| 07 | [07-guest-acl-router-fix.md](docs/logs/07-guest-acl-router-fix.md) | **Final guest fix** — router subinterface |
| 08 | [08-security-hardening.md](docs/logs/08-security-hardening.md) | SSH, port security, DHCP snooping, SERVER_ACCESS |
| 09 | [09-dhcp-snooping-pt-workaround.md](docs/logs/09-dhcp-snooping-pt-workaround.md) | DHCP snooping disabled for PT compatibility |

---

## Suggested learning path

1. Open the `.pkt` in Packet Tracer → **Logical** view → trace ISP → Router → Core → IDFs.
2. Read this README and [docs/GROUP-REFERENCE.md](docs/GROUP-REFERENCE.md).
3. Pick one department (e.g. IT on IDF 2-A): find its VLAN, ports, and gateway in [ports-and-cabling.md](docs/design/ports-and-cabling.md).
4. From a staff PC: ping gateway → DNS (`192.168.100.2`) → another department → internet (`10.0.0.1`).
5. From the guest laptop: confirm internal pings fail and internet works.
6. SSH to the core (`192.168.200.1`) and an IDF from the IT PC.
7. Read logs [07](docs/logs/07-guest-acl-router-fix.md) and [08](docs/logs/08-security-hardening.md) for the hardest Packet Tracer workarounds.

---

## Contributing / extending

When making changes:

1. Read [cisco.mdc](cisco.mdc) for hard rules and quick reference.
2. Check [docs/PROGRESS.md](docs/PROGRESS.md) and the latest log before editing the live `.pkt`.
3. After a major milestone: update PROGRESS.md and add a new file under `docs/logs/`.

---

*Built reality in logs wins over design docs. When in doubt, check PROGRESS.md and the latest session log.*
