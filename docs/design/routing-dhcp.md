# Routing & DHCP

[← Source of truth](../../cisco.mdc)

## Core switch (3560)

```
ip routing
ip route 0.0.0.0 0.0.0.0 192.168.200.14
```

SVIs — one per VLAN (10, 20, 30, 40, 50, 70, 80, 100, 200). **Vlan90 is shutdown** (guest uses router subif).

Example:

```
interface Vlan10
 ip address 192.168.10.1 255.255.255.192
 no shutdown
```

Repeat with correct mask per [vlans-and-ip.md](vlans-and-ip.md).

## Edge router (2911)

Physical `Gi0/0` is a **trunk** to core Fa0/8 (VLANs 90, 200). Subinterfaces:

```
interface GigabitEthernet0/0.200
 encapsulation dot1Q 200
 ip address 192.168.200.14 255.255.255.240

interface GigabitEthernet0/0.90
 encapsulation dot1Q 90
 ip address 192.168.90.1 255.255.255.128
 ip access-group GUEST_RESTRICT in
```

```
ip route 0.0.0.0 0.0.0.0 10.0.0.1

ip route 192.168.10.0 255.255.255.192 192.168.200.1
ip route 192.168.20.0 255.255.255.192 192.168.200.1
ip route 192.168.30.0 255.255.255.128 192.168.200.1
ip route 192.168.40.0 255.255.255.128 192.168.200.1
ip route 192.168.50.0 255.255.255.192 192.168.200.1
ip route 192.168.70.0 255.255.255.192 192.168.200.1
ip route 192.168.80.0 255.255.255.192 192.168.200.1
ip route 192.168.100.0 255.255.255.240 192.168.200.1
```

## DHCP — staff pools (core)

Excluded addresses first, then pools. All staff pools use `dns-server 192.168.100.2`.

```
ip dhcp excluded-address 192.168.10.1  192.168.10.5
! … repeat per VLAN gateway …
ip dhcp excluded-address 192.168.200.1 192.168.200.10

ip dhcp pool EXEC_POOL
 network 192.168.10.0 255.255.255.192
 default-router 192.168.10.1
 dns-server 192.168.100.2
```

Staff pools: `EXEC_POOL`, `FINANCE_POOL`, `OPS_POOL`, `CUSTSERV_POOL`, `IT_POOL`, `HR_POOL`, `MARKETING_POOL`.

> VLANs 100 and 200 are static only. Staff DHCP on core SVIs handles requests from access VLANs without `ip helper-address` on L2 IDFs.

## DHCP — guest pool (edge router)

```
ip dhcp excluded-address 192.168.90.1 192.168.90.10

ip dhcp pool GUEST_POOL
 network 192.168.90.0 255.255.255.128
 default-router 192.168.90.1
 dns-server 8.8.8.8
```

Guest DHCP is **not** on the core — it runs on Gi0/0.90. See [guest-network.md](guest-network.md).
