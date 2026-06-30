# Security

[← Source of truth](../../cisco.mdc)

**Full implementation guide:** [docs/logs/08-security-hardening.md](../logs/08-security-hardening.md)

**Lab credentials:** `admin` / `TamacAdmin2024!` · domain `tamac.local`

---

## ACL — Guest (Vlan90)

**Design target:** `GUEST_RESTRICT` inbound on Core `interface Vlan90`.

**Packet Tracer reality:** 3560-24PS often **does not save** `ip access-group` on SVIs. **Implemented:** guest gateway + ACL on **Edge Router Gi0/0.90** — [07-guest-acl-router-fix.md](../logs/07-guest-acl-router-fix.md).

### ACL definition

```
ip access-list extended GUEST_RESTRICT
 permit udp any any eq bootps
 permit udp any any eq bootpc
 deny   ip 192.168.90.0 0.0.0.127 192.168.10.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.20.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.30.0 0.0.0.127
 deny   ip 192.168.90.0 0.0.0.127 192.168.40.0 0.0.0.127
 deny   ip 192.168.90.0 0.0.0.127 192.168.50.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.70.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.80.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.100.0 0.0.0.15
 deny   ip 192.168.90.0 0.0.0.127 192.168.200.0 0.0.0.15
 permit ip 192.168.90.0 0.0.0.127 any
```

**Applied on Edge Router:**

```
interface GigabitEthernet0/0.90
 ip access-group GUEST_RESTRICT in
```

---

## ACL — Server (Vlan100, inbound)

```
ip access-list extended SERVER_ACCESS
 permit ip 192.168.100.0 0.0.0.15 192.168.100.0 0.0.0.15
 permit ip 192.168.50.0 0.0.0.63  192.168.100.0 0.0.0.15
 permit ip 192.168.20.0 0.0.0.63  192.168.100.0 0.0.0.15
 permit ip 192.168.30.0 0.0.0.127 192.168.100.0 0.0.0.15
 permit ip 192.168.10.0 0.0.0.63  192.168.100.0 0.0.0.15
 permit ip 192.168.40.0 0.0.0.127 192.168.100.0 0.0.0.15
 permit ip 192.168.70.0 0.0.0.63  192.168.100.0 0.0.0.15
 permit ip 192.168.80.0 0.0.0.63  192.168.100.0 0.0.0.15
 deny   ip any 192.168.100.0 0.0.0.15
 permit ip any any

interface Vlan100
 ip access-group SERVER_ACCESS in
```

> Same-subnet permit **first**. If PT won't bind to Vlan100, ACL still filters when applied — verify with `show running-config`.

---

## SSH (all devices)

```
ip domain-name tamac.local
crypto key generate rsa
1024
ip ssh version 2
username admin privilege 15 secret TamacAdmin2024!
enable secret TamacAdmin2024!

line vty 0 4
 login local
 transport input ssh
```

**IDF management (VLAN 200):**

| IDF | IP |
|---|---|
| 1-A | 192.168.200.2 |
| 1-B | 192.168.200.3 |
| 2-A | 192.168.200.4 |
| 2-B | 192.168.200.5 |

Each IDF: `ip default-gateway 192.168.200.1`

---

## Port security (IDF access ports only — not Fa0/23–24)

### Wired PC ports

```
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation shutdown
```

| IDF | PC ports |
|---|---|
| 1-A | Fa0/1–16 (not AP ports 17–19) |
| 1-B | Fa0/1–20 (not AP ports 21–22) |
| 2-A | Fa0/1–8, Fa0/15–19 (not AP ports 20–21) |
| 2-B | Fa0/1–15 (not AP port 16) |

### AP uplink ports (wireless — multiple MACs per port)

APs aggregate every wireless client onto one switch port. **Do not** use `maximum 1` on AP ports.

| AP port | IDF | `maximum` | `violation` |
|---|---|---|---|
| Fa0/17 (Guest) | 1-A | **50** | **restrict** |
| Fa0/18–19 (EXEC, FIN) | 1-A | 10 | restrict |
| Fa0/21–22 (CS, OPS) | 1-B | 10 | restrict |
| Fa0/20–21 (IT, HR) | 2-A | 10 | restrict |
| Fa0/16 (MARK) | 2-B | 10 | restrict |

```
interface FastEthernet0/17
 switchport port-security maximum 50
 switchport port-security violation restrict
```

Full CLI + troubleshooting: [log 10](../logs/10-ap-port-security-multi-client.md)

### Hacker PC wire-swap demo (wired ports)

Used in capstone defense to show **rogue device blocking**:

1. Legitimate PC connected first → sticky MAC learned.
2. Attacker unplugs that cable and connects a spare **`HACK-<dept>`** PC to the same switch port.
3. Port goes **err-disabled** (link red) — violation `shutdown`.
4. Reconnect original PC + `shutdown` / `no shutdown` on the switch port → link green, DHCP works.

Step-by-step for every department: [log 11](../logs/11-port-security-hacker-pc-demo.md).

---

## DHCP snooping

**Global:**

```
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,40,50,70,80,90,100,200
```

**Trust uplinks only:**

| Device | Trusted ports |
|---|---|
| CORE-SW | Fa0/8, Po1–Po4 |
| IDF 1-A | Po1, Fa0/23–24 |
| IDF 1-B | Po2, Fa0/23–24 |
| IDF 2-A | Po3, Fa0/23–24 |
| IDF 2-B | Po4, Fa0/23–24 |

---

## Checklist

| Feature | Where | Status |
|---|---|---|
| GUEST_RESTRICT | Edge Router Gi0/0.90 | ✅ Verified |
| SERVER_ACCESS | Core Vlan100 | Apply in session 08 |
| SSH v2 | Core, Router, IDFs | Session 08 |
| Port security | IDF access ports | Session 08 |
| DHCP snooping | Core + IDFs | Session 08 |
| VLAN 999 + shutdown | Unused ports | ✅ Done |
