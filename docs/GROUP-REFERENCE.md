# TAMAC Inc. — Group Reference & Things to Remember

> **Start here if you're new to the project.** This file gives full context for teammates learning the network structure, Packet Tracer quirks, and what was actually built (not just what the original design doc says).

**Related:** [cisco.mdc](../cisco.mdc) (index) · [PROGRESS.md](PROGRESS.md) (status) · [logs/](logs/) (step-by-step CLI) · [design/](design/) (topic deep-dives)

**Last updated:** 2025-06-25 · **Status:** ~100% complete

---

## 1. What this project is

TAMAC Inc. is a fictional e-commerce aggregator with a **two-story HQ** and **200–250 employees**. The capstone is a **secure, highly available enterprise LAN** built in **Cisco Packet Tracer**.

| Item | Value |
|---|---|
| Platform | Cisco Packet Tracer |
| Edge device | Cisco 2911 (Edge Router) |
| Core | Cisco 3560-24PS |
| Access | 4× Cisco 2960-24TT (IDF 1-A, 1-B, 2-A, 2-B) |
| WAN | Serial to ISP-Router (2911) — not Cloud-PT |
| DNS | Server-PT at `192.168.100.2` |

---

## 2. How to read this repo

```
NetAD/
├── cisco.mdc              ← Read FIRST (index + hard rules)
├── docs/
│   ├── PROGRESS.md        ← What's done / deviations / test results
│   ├── GROUP-REFERENCE.md ← This file (onboarding for teammates)
│   ├── deliverables/      ← Capstone submission package (5 deliverables)
│   ├── design/            ← Topic docs (VLANs, security, ports, etc.)
│   └── logs/              ← Session-by-session commands + verified output
```

**Rule of thumb:** If design docs conflict with `PROGRESS.md` or `logs/`, **trust what was built and logged**.

---

## 3. Hard rules — never break these

1. **No Procurement department** — never VLAN 60, `192.168.60.0/26`, or Procurement ports.
2. **Eight departments only:** Executive, IT, Operations, Finance, Customer Service, Marketing, HR, Guest.
3. **IDF 2-A Fa0/9–14** stay shutdown on VLAN 999 (unused — no Procurement).
4. **Built reality wins** over original design when they disagree.

---

## 4. Network topology (as built)

```
[ ISP-Router 2911 ] --Serial 10.0.0.0/30-- [ Edge Router 2911 ]
                                                | Gi0/0 (trunk .200 + .90)
                                           [ CORE-SW 3560 ]
                                            /  |  |  |  \
                                      Fa0/1  Po1 Po2 Po3 Po4
                                        |     |   |   |   |
                                    DNS-SVR IDF  IDF IDF IDF
                                            1-A 1-B 2-A 2-B
```

### MDF (Floor 2 — IT Server Room)

- Edge Router, CORE-SW, DNS-SVR only.

### IDF closets

| IDF | Floor | EtherChannel | Core uplink ports |
|---|---|---|---|
| IDF 1-A | 1 | Po1 | Core Gi0/1–2 |
| IDF 1-B | 1 | Po2 | Core Fa0/2–3 |
| IDF 2-A | 2 | Po3 | Core Fa0/4–5 |
| IDF 2-B | 2 | Po4 | Core Fa0/6–7 |

Router connects to Core **Fa0/8** (trunk VLANs 90 + 200). DNS connects to Core **Fa0/1** (access VLAN 100).

---

## 5. VLANs and gateways

| VLAN | Department / use | Subnet | Gateway |
|---|---|---|---|
| 10 | Executive | 192.168.10.0/26 | 192.168.10.1 |
| 20 | Finance | 192.168.20.0/26 | 192.168.20.1 |
| 30 | Operations | 192.168.30.0/25 | 192.168.30.1 |
| 40 | Customer Service | 192.168.40.0/25 | 192.168.40.1 |
| 50 | IT | 192.168.50.0/26 | 192.168.50.1 |
| 70 | HR | 192.168.70.0/26 | 192.168.70.1 |
| 80 | Marketing | 192.168.80.0/26 | 192.168.80.1 |
| 90 | Guest | 192.168.90.0/25 | **192.168.90.1 (Edge Router Gi0/0.90)** |
| 100 | Servers (DNS) | 192.168.100.0/28 | 192.168.100.1 |
| 200 | Management | 192.168.200.0/28 | 192.168.200.1 |
| 999 | Unused / shutdown | — | — |

**DNS for all DHCP pools:** `192.168.100.2` (except guest — guest uses `8.8.8.8` via router).

---

## 6. Key device IPs

| Device | IP | Notes |
|---|---|---|
| CORE-SW (Vlan200 SVI) | 192.168.200.1/28 | STP root, inter-VLAN routing, DHCP |
| Edge Router Gi0/0.200 | 192.168.200.14/28 | **Not .254** — invalid on /28 |
| Edge Router Gi0/0.90 | 192.168.90.1/25 | Guest gateway + GUEST_RESTRICT ACL |
| Edge Router WAN | 10.0.0.2/30 | NAT to ISP at 10.0.0.1 |
| DNS-SVR | 192.168.100.2/28 | A records + DNS service ON |
| IDF 1-A mgmt | 192.168.200.2/28 | Vlan200 + `ip default-gateway 192.168.200.1` |
| IDF 1-B mgmt | 192.168.200.3/28 | Same pattern |
| IDF 2-A mgmt | 192.168.200.4/28 | Same pattern |
| IDF 2-B mgmt | 192.168.200.5/28 | Same pattern |

---

## 7. Wi‑Fi SSIDs

| SSID | VLAN | Who uses it |
|---|---|---|
| `TAMAC-Corp-exec` | 10 | Executive staff laptops — WPA2-PSK |
| `TAMAC-Corp-fin` | 20 | Finance staff laptops |
| `TAMAC-Corp-ops` | 30 | Operations staff laptops |
| `TAMAC-Corp-cs` | 40 | Customer Service staff laptops |
| `TAMAC-Corp-it` | 50 | IT staff laptops |
| `TAMAC-Corp-hr` | 70 | HR staff laptops |
| `TAMAC-Corp-mark` | 80 | Marketing staff laptops |
| `TAMAC-Guest` | 90 | Guest laptop only — lobby AP on IDF 1-A Fa0/17 |

> Corporate Wi‑Fi uses **department-specific SSIDs** so each device auto-joins the right VLAN. All corporate SSIDs share passphrase `TAMACstaff2024!`.

**Guest laptop must show `192.168.90.x` gateway `192.168.90.1`.** If it gets a dept IP, it's on the wrong SSID.

---

## 8. Credentials (lab only)

| Setting | Value |
|---|---|
| Username | `admin` |
| Password | `TamacAdmin2024!` |
| Enable secret | `TamacAdmin2024!` |
| Domain | `tamac.local` |
| **IDF 2-B exception** | Password `Tamac2024` (PT quirk with `!` in secrets) |

SSH from IT PC:

```
ssh -l admin 192.168.200.1    ! Core
ssh -l admin 192.168.200.14   ! Edge Router
ssh -l admin 192.168.200.2      ! IDF 1-A
ssh -l admin 192.168.200.3    ! IDF 1-B
ssh -l admin 192.168.200.4    ! IDF 2-A
ssh -l admin 192.168.200.5    ! IDF 2-B (use Tamac2024)
```

Telnet should **fail** on all devices (`transport input ssh` only).

---

## 9. Critical deviations from original design

These are intentional — memorize them before changing anything:

| Topic | Original design | What we actually built |
|---|---|---|
| Core Gigabit ports | Gi0/1–9 | 3560-24PS has **only 2** Gi ports; uplinks use Fa0/2–7 + Gi0/1–2 |
| Router LAN IP | 192.168.200.254/28 | **192.168.200.14/28** (.254 out of range on /28) |
| Router → Core link | Access VLAN 200 | **Trunk** VLANs 90, 200 + router subinterfaces |
| Guest gateway | Core Vlan90 SVI | **Edge Router Gi0/0.90** — Core Vlan90 shutdown |
| GUEST_RESTRICT ACL | Core Vlan90 inbound | **Edge Router Gi0/0.90** inbound |
| Guest DHCP | Core GUEST_POOL | **Edge Router** pool on Gi0/0.90 |
| ISP | Cloud-PT | **ISP-Router (2911)** — Cloud lacks clock rate in PT |
| SERVER_ACCESS ACL | Core Vlan100 inbound | ACL defined on core; PT may not show bind on SVI — tests still pass |
| DHCP snooping trust | Port-channel interfaces | **Physical uplink ports** only (PT limitation) |

---

## 10. Security features (summary)

| Feature | Where | What it does |
|---|---|---|
| **GUEST_RESTRICT** | Edge Router Gi0/0.90 | Blocks guest from all internal subnets; allows internet via NAT |
| **SERVER_ACCESS** | CORE-SW (Vlan100) | Permits dept VLANs → DNS; denies everyone else → server subnet |
| **SSH v2** | Core, Router, all 4 IDFs | Encrypted management; telnet disabled |
| **Port security** | IDF access ports | Max 1 MAC, sticky, violation shutdown |
| **DHCP snooping** | Core + all IDFs | Trust uplinks only; blocks rogue DHCP on access ports |
| **EtherChannel + RSTP** | Core ↔ each IDF | HA uplinks; unplug one link, traffic still flows |

### Port security ranges (do NOT apply to Fa0/23–24 uplinks)

| IDF | Secured ports |
|---|---|
| IDF 1-A | Fa0/1–19 |
| IDF 1-B | Fa0/1–22 |
| IDF 2-A | Fa0/1–8, Fa0/15–21 |
| IDF 2-B | Fa0/1–17 |

### DHCP snooping trusted ports

| Device | Trusted interfaces |
|---|---|
| CORE-SW | Fa0/1, Fa0/2–8, Gi0/1–2 |
| Each IDF | Fa0/23–24 |

---

## 11. Packet Tracer quirks — read before troubleshooting

These caused real bugs during the build. Don't waste hours re-discovering them.

### SVI ACL binding fails silently

On the 3560, `ip access-group` on `interface Vlan90` or `Vlan100` may **accept in CLI but not save**. `show ip interface vlan X` shows **Inbound access list is not set**.

**Workarounds used:**
- **Guest:** Move L3 gateway + ACL to **Edge Router subinterface** Gi0/0.90 ([log 07](logs/07-guest-acl-router-fix.md))
- **SERVER_ACCESS:** ACL exists; functional tests pass; guest DNS block also enforced by router ACL

### Port-channel won't take DHCP snooping trust

`ip dhcp snooping trust` on `Port-channel1–4` returns **Invalid input** on 3560 in PT.

**Fix:** Trust the **physical member ports** (Gi0/1–2, Fa0/2–7 on core; Fa0/23–24 on IDFs).

### `operational on following VLANs: none`

Common PT display after enabling DHCP snooping. If global snooping is enabled and uplinks are trusted, you're fine for the defense demo.

### SSH password with special characters

IDF 2-B rejected `TamacAdmin2024!` but accepted `Tamac2024`. If SSH says "invalid login" on an IDF, recreate user with a simpler password or use the **Config tab → User Accounts**.

### ACL line order matters

Editing an existing ACL **appends** new lines at the bottom. Always `no ip access-list extended NAME` and recreate in correct order. **Permit same-subnet before deny** on SERVER_ACCESS.

### Common CLI typos

- Use `show`, not `how`
- Don't type `Password:` at the `#` prompt after enable already succeeded

---

## 12. Defense screenshots (saved)

Screenshots from final verification are in [deliverables/03-screenshots/](deliverables/03-screenshots/). Full index: [deliverables/README.md](deliverables/README.md).

### Connectivity ([03-screenshots/02-connectivity/](deliverables/03-screenshots/02-connectivity/))

| File | What it proves |
|---|---|
| [01-exec-pc-dhcp.png](deliverables/03-screenshots/02-connectivity/01-exec-pc-dhcp.png) | Executive PC DHCP on VLAN 10 |
| [02-exec-ping-dns.png](deliverables/03-screenshots/02-connectivity/02-exec-ping-dns.png) | Staff reaches DNS `192.168.100.2` |
| [03-exec-ping-finance-intervlan.png](deliverables/03-screenshots/02-connectivity/03-exec-ping-finance-intervlan.png) | Inter-VLAN routing works |
| [04-exec-ping-internet.png](deliverables/03-screenshots/02-connectivity/04-exec-ping-internet.png) | Staff internet via NAT |

### Guest ([03-screenshots/03-guest/](deliverables/03-screenshots/03-guest/))

| File | What it proves |
|---|---|
| [01-guest-laptop-dhcp.png](deliverables/03-screenshots/03-guest/01-guest-laptop-dhcp.png) | Guest DHCP on VLAN 90 |
| [02-guest-isolation-blocked.png](deliverables/03-screenshots/03-guest/02-guest-isolation-blocked.png) | Guest blocked from internal subnets |
| [03-guest-ping-internet.png](deliverables/03-screenshots/03-guest/03-guest-ping-internet.png) | Guest internet works |

### Security ([03-screenshots/04-security/](deliverables/03-screenshots/04-security/))

| File | What it proves |
|---|---|
| [01-it-ssh-core-sw.png](deliverables/03-screenshots/04-security/01-it-ssh-core-sw.png) | SSH from IT PC to CORE-SW |
| [02-it-ssh-idf2b.png](deliverables/03-screenshots/04-security/02-it-ssh-idf2b.png) | SSH from IT PC to IDF 2-B (`192.168.200.5`) |
| [03-idf1a-port-security.png](deliverables/03-screenshots/04-security/03-idf1a-port-security.png) | Port security on IDF 1-A access ports |

### High availability ([03-screenshots/05-ha/](deliverables/03-screenshots/05-ha/))

| File | What it proves |
|---|---|
| [01-core-etherchannel-summary.png](deliverables/03-screenshots/05-ha/01-core-etherchannel-summary.png) | EtherChannel Po1–Po4 up on core |

### Defense test checklist

| # | Test | From | Command | Expected |
|---|---|---|---|---|
| 1 | SSH to core | IT PC | `ssh -l admin 192.168.200.1` | Login OK |
| 2 | SSH to IDF | IT PC | `ssh -l admin 192.168.200.5` | Login OK (`Tamac2024`) |
| 3 | Telnet blocked | IT PC | `telnet 192.168.200.1` | Fail |
| 4 | Staff → DNS | IT PC | `ping 192.168.100.2` | 4/4 OK |
| 5 | Guest → DNS | Guest laptop | `ping 192.168.100.2` | Unreachable from 192.168.90.1 |
| 6 | Guest isolation | Guest laptop | `ping 192.168.10.1` | Unreachable from 192.168.90.1 |
| 7 | Guest internet | Guest laptop | `ping 10.0.0.1` | OK |
| 8 | Port security | Swap PC on secured port | — | Port err-disabled |
| 9 | EtherChannel HA | Unplug one uplink cable | PCs still ping | OK |
| 10 | DHCP snooping | CORE-SW | `show ip dhcp snooping` | Configured; may be disabled in PT — see [log 09](logs/09-dhcp-snooping-pt-workaround.md) |

---

## 13. Session logs — learn in order

| # | File | What you'll learn |
|---|---|---|
| 01 | [01-topology-core-idf-dns.md](logs/01-topology-core-idf-dns.md) | Base topology, core VLANs/SVIs/DHCP, all 4 IDFs, DNS |
| 02 | [02-wan-isp-router.md](logs/02-wan-isp-router.md) | ISP-Router, default route, NAT |
| 03 | [03-pc5-idf-dhcp.md](logs/03-pc5-idf-dhcp.md) | End-device DHCP recovery |
| 04 | [04-guest-ap-dhcp.md](logs/04-guest-ap-dhcp.md) | Guest AP, early ACL work |
| 05 | [05-end-devices-staff-wifi.md](logs/05-end-devices-staff-wifi.md) | PCs, APs, corporate Wi‑Fi, port maps |
| 06 | [06-guest-acl-fix.md](logs/06-guest-acl-fix.md) | Why guest isolation failed on core SVI |
| 07 | [07-guest-acl-router-fix.md](logs/07-guest-acl-router-fix.md) | **Final guest fix** — router subinterface approach |
| 08 | [08-security-hardening.md](logs/08-security-hardening.md) | SSH, SERVER_ACCESS, port security, DHCP snooping |
| 09 | [09-dhcp-snooping-pt-workaround.md](logs/09-dhcp-snooping-pt-workaround.md) | DHCP snooping PT workaround; deliverables capture |
| 10 | [10-ap-port-security-multi-client.md](logs/10-ap-port-security-multi-client.md) | **AP port-security** — multi-laptop guest/staff Wi‑Fi fix |

---

## 14. Topic docs (when you need detail)

| Topic | File |
|---|---|
| Company + hardware | [design/overview.md](design/overview.md) |
| VLANs + subnets | [design/vlans-and-ip.md](design/vlans-and-ip.md) |
| Routing + DHCP pools | [design/routing-dhcp.md](design/routing-dhcp.md) |
| Ports + cabling map | [design/ports-and-cabling.md](design/ports-and-cabling.md) |
| End devices + AP ports | [design/end-devices.md](design/end-devices.md) |
| Corporate Wi‑Fi | [design/corporate-wifi.md](design/corporate-wifi.md) |
| Guest network | [design/guest-network.md](design/guest-network.md) |
| Security (ACLs, SSH) | [design/security.md](design/security.md) |
| EtherChannel + STP | [design/high-availability.md](design/high-availability.md) |
| DNS server setup | [design/dns.md](design/dns.md) |
| PT tips + checklist | [design/packet-tracer-guide.md](design/packet-tracer-guide.md) |

---

## 15. Quick troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Guest reaches internal VLANs | Wrong SSID or ACL not on router Gi0/0.90 | `ipconfig` → must be 90.x; check [log 07](logs/07-guest-acl-router-fix.md) |
| `show ip interface vlan X` ACL not set | PT SVI bug | Guest: use router subif; DNS: verify functional ping tests |
| SSH invalid login on IDF | Password special char bug | Use `Tamac2024` or Config tab user |
| DHCP snooping trust fails on Po | PT limitation | Trust Fa0/23–24 (IDF) or Gi0/1–2 + Fa0/2–7 (core) |
| Port shut down (red) | Port security violation | `shutdown` / `no shutdown` on port; reconnect original device |
| Core can't reach DNS after ACL | Deny before same-subnet permit | Recreate SERVER_ACCESS with 100.0/28 permit **first** |
| AP link red / err-disabled | Port security MAC violation on AP port | Raise `maximum` (guest 50, staff 10), `violation restrict` — [log 10](logs/10-ap-port-security-multi-client.md) |
| Guest DHCP fails, pool not full | AP port MAC limit hit | Same as above on IDF 1-A Fa0/17 |
| No DHCP on PCs after snooping | Option 82 on untrusted Po | `no ip dhcp snooping` on core + IDFs — [log 09](logs/09-dhcp-snooping-pt-workaround.md) |
| Router LAN .254 doesn't work | /28 only allows .1–.14 | Use **192.168.200.14** |

---

## 16. Suggested learning path for new groupmates

1. Open the `.pkt` file and switch to **Logical** view — trace ISP → Router → Core → IDFs.
2. Read this file + [PROGRESS.md](PROGRESS.md).
3. Pick one department (e.g. IT on IDF 2-A): find its VLAN, gateway, IDF ports in [ports-and-cabling.md](design/ports-and-cabling.md).
4. From **PC-IT**, ping gateway → DNS → another dept → internet. Confirm each hop.
5. From **Guest laptop**, confirm isolation (internal fail, internet OK).
6. SSH to one core device and one IDF — practice management VLAN 200.
7. Read [log 07](logs/07-guest-acl-router-fix.md) and [log 08](logs/08-security-hardening.md) — these explain the hardest PT workarounds.

---

*Built reality in logs wins over design docs. When in doubt, check PROGRESS.md and the latest session log before changing anything.*
