# Session 08 — Security Hardening

**Date:** 2025-06-24  
**Status:** Implementation guide

**Prerequisites:** Network functional; guest ACL on Edge Router (session 07).

**Credentials (lab use):**

| Setting | Value |
|---|---|
| Username | `admin` |
| Password | `TamacAdmin2024!` (IDF-2B: `Tamac2024` — PT quirk) |
| Domain | `tamac.local` |

### Part 1 status — SSH ✅ complete

| Device | Mgmt IP | SSH from IT PC |
|---|---|---|
| CORE-SW | 192.168.200.1 | ✅ |
| Edge Router | 192.168.200.14 | ✅ |
| IDF 1-A | 192.168.200.2 | ✅ |
| IDF 1-B | 192.168.200.3 | ✅ |
| IDF 2-A | 192.168.200.4 | ✅ |
| IDF 2-B | 192.168.200.5 | ✅ (`Tamac2024`) |

---

## Part 1 — SSH on all devices

Apply to **CORE-SW**, **Edge Router**, **IDF 1-A**, **IDF 1-B**, **IDF 2-A**, **IDF 2-B**.

```
enable
configure terminal
ip domain-name tamac.local
crypto key generate rsa
1024
ip ssh version 2
username admin privilege 15 secret TamacAdmin2024!
enable secret TamacAdmin2024!
line vty 0 4
 login local
 transport input ssh
end
```

> When prompted for RSA key size, type **1024** (Packet Tracer may not accept 2048). Press Enter to confirm.

**Optional — ISP-Router:** same block if you want SSH there too.

### Management IPs for IDF SSH (VLAN 200)

**IDF 1-A:**

```
configure terminal
interface Vlan200
 ip address 192.168.200.2 255.255.255.240
 no shutdown
ip default-gateway 192.168.200.1
end
```

**IDF 1-B:**

```
interface Vlan200
 ip address 192.168.200.3 255.255.255.240
 no shutdown
ip default-gateway 192.168.200.1
```

**IDF 2-A:**

```
interface Vlan200
 ip address 192.168.200.4 255.255.255.240
 no shutdown
ip default-gateway 192.168.200.1
```

**IDF 2-B:**

```
interface Vlan200
 ip address 192.168.200.5 255.255.255.240
 no shutdown
ip default-gateway 192.168.200.1
```

### Test SSH from IT PC

**Desktop** → **Command Prompt**:

```
ssh -l admin 192.168.200.1
ssh -l admin 192.168.200.4
```

Password: `TamacAdmin2024!`

**Telnet test (should fail):**

```
telnet 192.168.200.1
```

Connection refused or failed = good.

---

## Part 2 — SERVER_ACCESS ACL (DNS protection)

ACL already exists on CORE-SW. Apply to **Vlan100** (may require GUI if CLI won't stick — same PT quirk as guest).

**CORE-SW:**

```
enable
configure terminal
no ip access-list extended SERVER_ACCESS

ip access-list extended SERVER_ACCESS
 permit ip 192.168.100.0 0.0.0.15 192.168.100.0 0.0.0.15
 permit ip 192.168.50.0 0.0.0.63 192.168.100.0 0.0.0.15
 permit ip 192.168.20.0 0.0.0.63 192.168.100.0 0.0.0.15
 permit ip 192.168.30.0 0.0.0.127 192.168.100.0 0.0.0.15
 permit ip 192.168.10.0 0.0.0.63 192.168.100.0 0.0.0.15
 permit ip 192.168.40.0 0.0.0.127 192.168.100.0 0.0.0.15
 permit ip 192.168.70.0 0.0.0.63 192.168.100.0 0.0.0.15
 permit ip 192.168.80.0 0.0.0.63 192.168.100.0 0.0.0.15
 deny   ip any 192.168.100.0 0.0.0.15
 permit ip any any

interface Vlan100
 ip access-group SERVER_ACCESS in
end
```

Verify:

```
show access-lists SERVER_ACCESS
ping 192.168.100.2
```

From **IT PC**: `ping 192.168.100.2` OK. From **Guest laptop**: should fail (guest blocked from internal).

> Added permits for VLANs 10, 40, 70, 80 so all departments can reach DNS.

### Part 2 status — SERVER_ACCESS ✅ (functional)

| Check | Result |
|---|---|
| `show access-lists SERVER_ACCESS` | ✅ All 10 lines correct |
| IT PC → `ping 192.168.100.2` | ✅ OK |
| Guest → `ping 192.168.100.2` | ✅ Fail |
| `show ip interface vlan 100` inbound ACL | ⚠️ PT shows **not set** (same SVI quirk as old Vlan90 guest ACL) |

**Defense note:** Guest DNS block is enforced by **GUEST_RESTRICT** on Edge Router even if Core SVI won't bind the ACL. ACL is still in running config for documentation; optional GUI bind: Core **Config** → **INTERFACE** → **Vlan100** → Inbound **SERVER_ACCESS**.

---

## Part 3 — Port security (IDF access ports only)

**Do NOT** apply to **Fa0/23–24** or **Port-channel** uplinks.

### IDF 1-A (Fa0/1–16 wired PCs · Fa0/17–19 APs)

```
enable
configure terminal
interface range FastEthernet0/1 - 16
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
interface FastEthernet0/17
 switchport port-security maximum 50
 switchport port-security violation restrict
interface range FastEthernet0/18 - 19
 switchport port-security maximum 10
 switchport port-security violation restrict
end
```

### IDF 1-B (Fa0/1–14, Fa0/15–20 wired · Fa0/21–22 APs)

```
interface range FastEthernet0/1 - 14, FastEthernet0/15 - 20
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
interface range FastEthernet0/21 - 22
 switchport port-security maximum 10
 switchport port-security violation restrict
end
```

### IDF 2-A (Fa0/1–8, Fa0/15–19 wired · Fa0/20–21 APs)

```
interface range FastEthernet0/1 - 8, FastEthernet0/15 - 19
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
interface range FastEthernet0/20 - 21
 switchport port-security maximum 10
 switchport port-security violation restrict
end
```

### IDF 2-B (Fa0/1–12, Fa0/13–15, Fa0/17 wired · Fa0/16 AP)

```
interface range FastEthernet0/1 - 12, FastEthernet0/13 - 15, FastEthernet0/17
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
interface FastEthernet0/16
 switchport port-security maximum 10
 switchport port-security violation restrict
end
```

Full paste blocks: [port-security-all-departments-paste.txt](../deliverables/02-device-configs/port-security-all-departments-paste.txt)

### Verify

On any IDF:

```
show port-security
show port-security interface FastEthernet0/1
```

### Port security violation demo

Full hacker-PC wire-swap script (any department): [log 11](11-port-security-hacker-pc-demo.md).

1. Note MAC on **PC-IT** port (IDF 2-A Fa0/1).
2. Disconnect PC-IT, connect a **different PC** (e.g. **HACK-IT**) to same port.
3. Port should go **err-disabled** (red link).
4. Fix: reconnect **PC-IT**, then `shutdown` / `no shutdown` on that port.

---

## Part 4 — DHCP snooping

### CORE-SW

```
enable
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,40,50,70,80,100,200

interface FastEthernet0/1
 ip dhcp snooping trust
interface FastEthernet0/8
 ip dhcp snooping trust
interface GigabitEthernet0/1
 ip dhcp snooping trust
interface GigabitEthernet0/2
 ip dhcp snooping trust
interface range FastEthernet0/2 - 7
 ip dhcp snooping trust
end
```

> **PT quirk:** `ip dhcp snooping trust` fails on **Port-channel** on 3560 — trust **physical member ports** instead (Gi0/1–2, Fa0/2–7).  
> **Fa0/8** — Edge Router guest DHCP via trunk. **Fa0/1** — DNS (static; optional trust).

### Each IDF (same pattern)

**IDF 1-A through 2-B** — trust uplink physical ports only (not Port-channel):

```
enable
configure terminal
ip dhcp snooping
ip dhcp snooping vlan 10,20,30,40,50,70,80,90,100,200

interface range FastEthernet0/23 - 24
 ip dhcp snooping trust
end
```

### Part 4 status — DHCP snooping ✅ complete

| Device | Trusted ports |
|---|---|
| CORE-SW | Fa0/1, Fa0/2–8, Gi0/1–2 |
| IDF 1-A | Fa0/23–24 |
| IDF 1-B | Fa0/23–24 |
| IDF 2-A | Fa0/23–24 |
| IDF 2-B | Fa0/23–24 |

---

```
show ip dhcp snooping
show ip dhcp snooping binding
```

### Rogue DHCP demo (optional)

1. Place **Server-PT** on an **untrusted** IDF access port (e.g. spare port).
2. Enable DHCP on server.
3. PC on same VLAN should **not** get IP from rogue server if snooping works.

---

## Part 5 — Defense test checklist

| # | Test | From | Expected |
|---|---|---|---|
| 1 | SSH to core | IT PC | `ssh -l admin 192.168.200.1` OK |
| 2 | SSH to IDF 2-A | IT PC | `ssh -l admin 192.168.200.4` OK |
| 3 | Telnet blocked | IT PC | `telnet 192.168.200.1` fail |
| 4 | DNS from IT | PC-IT | `ping 192.168.100.2` OK |
| 5 | DNS from guest | Guest laptop | `ping 192.168.100.2` fail |
| 6 | Guest isolation | Guest laptop | `ping 192.168.10.1` unreachable |
| 7 | Guest internet | Guest laptop | `ping 10.0.0.1` OK |
| 8 | Port security | Rogue PC wrong port | Port err-disabled |
| 9 | EtherChannel HA | Unplug one uplink | PCs still ping |
| 10 | DHCP snooping | `show ip dhcp snooping` | Enabled, trust on uplinks |

---

## Troubleshooting

| Issue | Fix |
|---|---|
| SSH "no supported authentication" | Regenerate RSA key; use `ssh -l admin` |
| SSH invalid login (IDF only) | Recreate user with simple password (`Tamac2024`); or use Config tab |
| Can't SSH to IDF | Check Vlan200 IP + `ip default-gateway 192.168.200.1` |
| DHCP snooping trust on Po fails | Trust physical uplink ports (Core Gi0/1–2, Fa0/2–7; IDF Fa0/23–24) |
| Port security shut down port | `int fa0/x` → `shutdown` → `no shutdown` |
| DHCP broken after snooping | Verify uplink + port-channel **trust** |
| SERVER_ACCESS not applied | Check `show running-config` for `ip access-group` under Vlan100 |

---

## After completion

- **File → Save** `.pkt`
- Screenshot SSH + guest/staff tests
- Export `show running-config` from CORE-SW and Edge Router
