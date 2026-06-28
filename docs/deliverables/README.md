# TAMAC Inc. — Capstone deliverables

Submission package for the Packet Tracer network project. All paths are relative to this folder.

**Packet Tracer file:** `Final Proj.pkt` (repo root)

---

## Deliverable index

| # | Deliverable | Document | Screenshots / exports |
|---|---|---|---|
| 1 | Network diagram | [01-network-diagram/logical-topology-labeled.png](01-network-diagram/logical-topology-labeled.png) | [LABELS.md](01-network-diagram/LABELS.md) for label reference |
| 2 | Device configurations | Text exports below | Per-device screenshot folders |
| 3 | Screenshots (proof tests) | — | [03-screenshots/](03-screenshots/) |
| 4 | IP addressing tables | [04-ip-addressing.md](04-ip-addressing.md) | [04-ip-addressing-screenshots/](04-ip-addressing-screenshots/) |
| 5 | VLAN tables | [05-vlan-table.md](05-vlan-table.md) | [05-vlan-table-screenshots/](05-vlan-table-screenshots/) |
| 6 | **Verification testing (sequential)** | [06-verification-testing/README.md](06-verification-testing/README.md) | [06-verification-testing/screenshots/](06-verification-testing/screenshots/) |

---

## 6. Verification testing (sequential — complete in order)

Use this package for defense / rubric proof. **Each phase must pass before the next.**

| Phase | Guide | Screenshots |
|---|---|---|
| 1 Connectivity | [01-connectivity-testing.md](06-verification-testing/01-connectivity-testing.md) | `screenshots/01-connectivity/` |
| 2 Inter-VLAN | [02-inter-vlan-communication.md](06-verification-testing/02-inter-vlan-communication.md) | `screenshots/02-inter-vlan/` |
| 3 Routing | [03-routing-verification.md](06-verification-testing/03-routing-verification.md) | `screenshots/03-routing/` |
| 4 DHCP | [04-dhcp-verification.md](06-verification-testing/04-dhcp-verification.md) | `screenshots/04-dhcp/` |
| 5 Security (ACL, SSH, port sec, snooping, auth) | [05-security-testing.md](06-verification-testing/05-security-testing.md) | `screenshots/05-security/` |
| 6 Guest network | [06-guest-network-testing.md](06-verification-testing/06-guest-network-testing.md) | `screenshots/06-guest/` |

---

## 1. Network diagram

| File | Description |
|---|---|
| [logical-topology-labeled.png](01-network-diagram/logical-topology-labeled.png) | Full labeled logical topology |
| [logical-topology-unlabeled.png](01-network-diagram/logical-topology-unlabeled.png) | Before labels (archive) |
| [LABELS.md](01-network-diagram/LABELS.md) | Copy-paste labels for Packet Tracer notes |

---

## 2. Device configurations

### Text exports (`show running-config`)

| Device | File |
|---|---|
| CORE-SW | [core-sw-running-config.txt](02-device-configs/core-sw-running-config.txt) |
| Edge Router | [edge-router-running-config.txt](02-device-configs/edge-router-running-config.txt) |
| IDF 1-A | [idf-1a-running-config.txt](02-device-configs/idf-1a-running-config.txt) |
| IDF 1-B | [idf-1b-running-config.txt](02-device-configs/idf-1b-running-config.txt) |
| IDF 2-A | [idf-2a-running-config.txt](02-device-configs/idf-2a-running-config.txt) |
| IDF 2-B | [idf-2b-running-config.txt](02-device-configs/idf-2b-running-config.txt) |

### CLI screenshots

| Device | Folder |
|---|---|
| CORE-SW | [02-device-configs/core-sw/](02-device-configs/core-sw/) |
| Edge Router | [02-device-configs/edge-router/](02-device-configs/edge-router/) |
| IDF 1-A | [02-device-configs/idf-1a/](02-device-configs/idf-1a/) |
| IDF 1-B | [02-device-configs/idf-1b/](02-device-configs/idf-1b/) |
| IDF 2-A | [02-device-configs/idf-2a/](02-device-configs/idf-2a/) |
| IDF 2-B | [02-device-configs/idf-2b/](02-device-configs/idf-2b/) |

---

## 3. Screenshots (connectivity and security proof)

| Category | Folder | What it proves |
|---|---|---|
| Staff connectivity | [03-screenshots/02-connectivity/](03-screenshots/02-connectivity/) | DHCP, DNS, inter-VLAN, internet |
| Guest network | [03-screenshots/03-guest/](03-screenshots/03-guest/) | Guest DHCP, isolation, internet |
| Security | [03-screenshots/04-security/](03-screenshots/04-security/) | SSH to core and IDF, port security |
| High availability | [03-screenshots/05-ha/](03-screenshots/05-ha/) | EtherChannel summary |

---

## 4. IP addressing

| File | Description |
|---|---|
| [04-ip-addressing.md](04-ip-addressing.md) | Department subnets, DHCP ranges, static IPs, routes |
| [04-ip-addressing-screenshots/01-core-sw-svi-addresses.png](04-ip-addressing-screenshots/01-core-sw-svi-addresses.png) | Core SVI proof |
| [04-ip-addressing-screenshots/02-edge-router-interface-addresses.png](04-ip-addressing-screenshots/02-edge-router-interface-addresses.png) | Router WAN, mgmt, guest subifs |

---

## 5. VLAN tables

| File | Description |
|---|---|
| [05-vlan-table.md](05-vlan-table.md) | VLAN summary, port maps, wireless VLAN map |
| [05-vlan-table-screenshots/01-core-sw-vlan-brief.png](05-vlan-table-screenshots/01-core-sw-vlan-brief.png) | `show vlan brief` on core |
| [05-vlan-table-screenshots/02-idf1a-interface-vlan-assignments.png](05-vlan-table-screenshots/02-idf1a-interface-vlan-assignments.png) | IDF access port VLANs |
| [05-vlan-table-screenshots/03-idf1a-trunk-vlans.png](05-vlan-table-screenshots/03-idf1a-trunk-vlans.png) | Trunk VLAN forwarding |
| [05-vlan-table-screenshots/04-edge-router-guest-vlan90.png](05-vlan-table-screenshots/04-edge-router-guest-vlan90.png) | Guest gateway on router |

---

## Related documentation

| Need | Location |
|---|---|
| Project overview | [../../README.md](../../README.md) |
| Build status | [../PROGRESS.md](../PROGRESS.md) |
| Teammate onboarding | [../GROUP-REFERENCE.md](../GROUP-REFERENCE.md) |
| Session logs | [../logs/](../logs/) |
| Design deep-dives | [../design/](../design/) |
