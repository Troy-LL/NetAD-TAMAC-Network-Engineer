# Session 07 — Guest ACL via Edge Router (PT 3560 Workaround)

**Status:** Applied and verified — guest `ping 192.168.10.1` returns unreachable from 192.168.90.1

**Fix:** Apply guest L3 gateway + `GUEST_RESTRICT` on **Edge Router** (2911). Routers apply ACLs reliably in Packet Tracer.

**Keep:** Guest AP on IDF 1-A Fa0/17 (VLAN 90). Corporate unchanged.

---

## Option A — Trunk + subinterface (keeps AP on IDF) **Recommended**

### Step 1 — Core Fa0/8: access → trunk

**CORE-SW:**

```
enable
configure terminal
interface FastEthernet0/8
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 90,200
 no shutdown
end
```

### Step 2 — Remove guest SVI on core (router becomes guest gateway)

**CORE-SW:**

```
configure terminal
interface Vlan90
 no ip address
 shutdown
no ip dhcp pool GUEST_POOL
end
```

### Step 3 — Edge Router: subinterfaces + ACL + guest DHCP

**Edge Router** — paste in order:

```
enable
configure terminal
no ip access-list extended GUEST_RESTRICT

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

interface GigabitEthernet0/0
 no ip address
 no shutdown

interface GigabitEthernet0/0.200
 encapsulation dot1Q 200
 ip address 192.168.200.14 255.255.255.240
 ip nat inside

interface GigabitEthernet0/0.90
 encapsulation dot1Q 90
 ip address 192.168.90.1 255.255.255.128
 ip access-group GUEST_RESTRICT in
 ip nat inside

no ip route 192.168.90.0 255.255.255.128 192.168.200.1

ip dhcp excluded-address 192.168.90.1 192.168.90.5
ip dhcp pool GUEST_POOL
 network 192.168.90.0 255.255.255.128
 default-router 192.168.90.1
 dns-server 8.8.8.8

end
```

> Guest DNS uses `8.8.8.8` because ACL blocks internal `192.168.100.2`.

### Step 4 — Verify router

```
show ip interface brief
show ip access-lists GUEST_RESTRICT
ping 192.168.200.1
```

`Gi0/0.90` and `Gi0/0.200` should be up/up. `Gi0/0.90` should show ACL in.

### Step 5 — Guest laptop

Renew DHCP (Desktop → IP Configuration → DHCP). Must get **192.168.90.x**, gateway **192.168.90.1**.

```
ipconfig
ping 192.168.90.1
ping 192.168.10.1
ping 10.0.0.1
```

| Test | Expected |
|---|---|
| ping 192.168.90.1 | OK |
| ping 192.168.10.1 | **Fail** |
| ping 10.0.0.1 | OK |

---

## Option B — Guest AP on Router Gi0/1 (simpler cable, design deviation)

1. Move Guest AP **Port 0** from IDF 1-A Fa0/17 → **Edge Router Gi0/1**
2. **Edge Router:**

```
configure terminal
interface GigabitEthernet0/1
 ip address 192.168.90.1 255.255.255.128
 ip access-group GUEST_RESTRICT in
 ip nat inside
 no shutdown
ip dhcp excluded-address 192.168.90.1 192.168.90.5
ip dhcp pool GUEST_POOL
 network 192.168.90.0 255.255.255.128
 default-router 192.168.90.1
 dns-server 8.8.8.8
end
```

3. Shutdown IDF 1-A Fa0/17 or leave unused.

---

## PT 3560 limitation (for write-up)

Document in capstone report: *Catalyst 3560-24PS in Packet Tracer accepts `ip access-list` but does not persist `ip access-group` on SVIs. Guest isolation implemented on Edge Router where ACL binding is supported.*

---

## Revert (if needed)

**CORE-SW Fa0/8 back to access:**

```
interface FastEthernet0/8
 switchport mode access
 switchport access vlan 200
```

Restore `interface Vlan90` IP and core `GUEST_POOL` DHCP.
