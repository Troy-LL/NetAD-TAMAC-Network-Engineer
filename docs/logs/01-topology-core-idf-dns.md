# Session 01 — Topology, Core, Router, DNS, IDF Switches

**Date:** 2025-06-24  
**Status:** Successful — backbone layer complete

---

## 1. Logical Topology

```
[ Cloud-PT ISP ]
       | Serial DCE
[ 2911 Edge Router ]
       | Straight-Through (Gi0/0 → Core Fa0/8)
[ 3560-24PS CORE-SW ]
   |                    \
   | Straight-Through     \  (×2 EtherChannel per IDF)
[ Server-PT DNS-SVR ]     +-- IDF 1-A, 1-B, 2-A, 2-B (2960-24TT)
   (Core Fa0/1)
```

**Device labels in Packet Tracer:** ISP, Edge Router, Core switch, DNS-SVR, IDF 1-A / 1-B / 2-A / 2-B

---

## 2. Cabling Reference

| Connection | Cable | Ports |
|---|---|---|
| ISP → Edge Router | Serial DCE | Cloud Serial0 → Router Serial0/0/0 |
| Edge Router → Core | Copper Straight-Through | Router Gi0/0 → Core **Fa0/8** |
| DNS-SVR → Core | Copper Straight-Through | Server Fa0 → Core **Fa0/1** |
| IDF 1-A → Core | Straight-Through ×2 | Fa0/23–24 → Core **Gi0/1–2** |
| IDF 1-B → Core | Straight-Through ×2 | Fa0/23–24 → Core **Fa0/2–3** |
| IDF 2-A → Core | Straight-Through ×2 | Fa0/23–24 → Core **Fa0/4–5** |
| IDF 2-B → Core | Straight-Through ×2 | Fa0/23–24 → Core **Fa0/6–7** |

**Router note:** 2911 needs **HWIC-2T** (Physical tab, power off) before Serial0/0/0 appears.

---

## 3. Core Switch (CORE-SW) — Final Config Summary

### VLANs (no VLAN 60)

`10, 20, 30, 40, 50, 70, 80, 90, 100, 200, 999`

### Key interface assignments

| Port | Role |
|---|---|
| Fa0/1 | DNS-SVR access VLAN 100 |
| Fa0/8 | Edge Router access VLAN 200 |
| Gi0/1–2 | IDF 1-A EtherChannel 1 |
| Fa0/2–3 | IDF 1-B EtherChannel 2 |
| Fa0/4–5 | IDF 2-A EtherChannel 3 |
| Fa0/6–7 | IDF 2-B EtherChannel 4 |
| Fa0/9–24 | Shutdown VLAN 999 |

### SVIs

| VLAN | IP | Gateway for |
|---|---|---|
| 10 | 192.168.10.1/26 | Executive |
| 20 | 192.168.20.1/26 | Finance |
| 30 | 192.168.30.1/25 | Operations |
| 40 | 192.168.40.1/25 | Customer Service |
| 50 | 192.168.50.1/26 | IT |
| 70 | 192.168.70.1/26 | HR |
| 80 | 192.168.80.1/26 | Marketing |
| 90 | 192.168.90.1/25 | Guest |
| 100 | 192.168.100.1/28 | Servers |
| 200 | 192.168.200.1/28 | Management |

### Routing

```
ip routing
ip route 0.0.0.0 0.0.0.0 192.168.200.14
```

### ACLs (final working SERVER_ACCESS order)

```
ip access-list extended SERVER_ACCESS
 permit ip 192.168.100.0 0.0.0.15 192.168.100.0 0.0.0.15
 permit ip 192.168.50.0 0.0.0.63  192.168.100.0 0.0.0.15
 permit ip 192.168.20.0 0.0.0.63  192.168.100.0 0.0.0.15
 permit ip 192.168.30.0 0.0.0.127 192.168.100.0 0.0.0.15
 deny   ip any 192.168.100.0 0.0.0.15
 permit ip any any
```

> **Lesson:** Editing an existing ACL appends lines at the bottom. Use `no ip access-list extended SERVER_ACCESS` and recreate in order.

### Verified EtherChannel output (CORE-SW)

```
1      Po1(SU)    LACP   Gig0/1(P) Gig0/2(P)
2      Po2(SU)    LACP   Fa0/2(P) Fa0/3(P)
3      Po3(SU)    LACP   Fa0/4(P) Fa0/5(P)
4      Po4(SU)    LACP   Fa0/6(P) Fa0/7(P)
```

---

## 4. Edge Router — LAN Config

```
interface GigabitEthernet0/0
 ip address 192.168.200.14 255.255.255.240
 no shutdown
```

### Static routes (internal → core)

```
ip route 192.168.10.0 255.255.255.192 192.168.200.1
ip route 192.168.20.0 255.255.255.192 192.168.200.1
ip route 192.168.30.0 255.255.255.128 192.168.200.1
ip route 192.168.40.0 255.255.255.128 192.168.200.1
ip route 192.168.50.0 255.255.255.192 192.168.200.1
ip route 192.168.70.0 255.255.255.192 192.168.200.1
ip route 192.168.80.0 255.255.255.192 192.168.200.1
ip route 192.168.90.0 255.255.255.128 192.168.200.1
ip route 192.168.100.0 255.255.255.240 192.168.200.1
```

### Verified pings

```
CORE-SW# ping 192.168.200.14   → 80% (4/5)
Router# ping 192.168.200.1     → 100% (5/5)
CORE-SW# ping 192.168.100.2    → 100% (5/5) with ACL applied
```

---

## 5. DNS-SVR

| Setting | Value |
|---|---|
| IP | 192.168.100.2 |
| Subnet | 255.255.255.240 |
| Gateway | 192.168.100.1 |
| DNS | 192.168.100.2 |

**Pending:** DNS service A records in Config → SERVICES → DNS

---

## 6. IDF Access Switches

### IDF-1A (channel-group 1)

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–8, Fa0/18 | 10 | Executive PCs + AP |
| Fa0/9–16 | 20 | Finance PCs |
| Fa0/17 | 90 | Guest AP |
| Fa0/19–22 | 999 | Shutdown |
| Fa0/23–24 | Trunk | Po1 → Core |

**Verified:** `Po1(SU) Fa0/23(P) Fa0/24(P)`

### IDF-1B (channel-group 2)

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–14, Fa0/21 | 40 | Customer Service |
| Fa0/15–20, Fa0/22 | 30 | Operations |
| Fa0/23–24 | Trunk | Po2 → Core |

**Verified:** `Po2(SU) Fa0/23(P) Fa0/24(P)`

### IDF-2A (channel-group 3)

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–8, Fa0/20 | 50 | IT |
| Fa0/9–14 | 999 | Shutdown (unused — no Procurement dept) |
| Fa0/15–19, Fa0/21 | 70 | HR |
| Fa0/22 | 999 | Shutdown |
| Fa0/23–24 | Trunk | Po3 → Core |

**Verified:** `Po3(SU) Fa0/23(P) Fa0/24(P)`

### IDF-2B (channel-group 4)

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–12, Fa0/16 | 80 | Marketing |
| Fa0/13–15, Fa0/17 | 70 | HR Training |
| Fa0/18–22 | 999 | Shutdown |
| Fa0/23–24 | Trunk | Po4 → Core |

**Verified:** `Po4(SU) Fa0/23(P) Fa0/24(P)`

### Optional cleanup (harmless empty Po1 on 1-B, 2-A, 2-B)

```
no interface port-channel1
```

---

## 7. Packet Tracer CLI Tips (learned this session)

| Mistake | Fix |
|---|---|
| `vlan 10,20,30` on one line | One `vlan X` per line |
| `configure terminal` from `>` | `en` first |
| `config tg` | `config t` |
| `how` instead of `show` | `show` |
| Appending to existing ACL | `no ip access-list extended NAME` then recreate |

---

## 8. Not Yet Done (carried to next session)

- [ ] WAN Serial0/0/0 + ISP Cloud config
- [ ] NAT/PAT on router
- [ ] DNS A records
- [ ] PCs and DHCP testing
- [ ] APs
- [ ] SSH, port security, DHCP snooping
- [ ] GUEST_RESTRICT ACL verification with guest device
