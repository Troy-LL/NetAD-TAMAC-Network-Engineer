# Routing & DHCP

[← Source of truth](../../cisco.mdc)

## Core switch (3560)

```
ip routing
ip route 0.0.0.0 0.0.0.0 192.168.200.14
```

SVIs — one per VLAN (10, 20, 30, 40, 50, 70, 80, 90, 100, 200). Example:

```
interface Vlan10
 ip address 192.168.10.1 255.255.255.192
 no shutdown
```

Repeat with correct mask per [vlans-and-ip.md](vlans-and-ip.md).

## Edge router (2911)

```
interface GigabitEthernet0/0
 ip address 192.168.200.14 255.255.255.240
 no shutdown

ip route 0.0.0.0 0.0.0.0 10.0.0.1

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

## DHCP (on core only)

Excluded addresses first, then pools. All pools use `dns-server 192.168.100.2`.

```
ip dhcp excluded-address 192.168.10.1  192.168.10.5
! … repeat per VLAN gateway …
ip dhcp excluded-address 192.168.200.1 192.168.200.10

ip dhcp pool EXEC_POOL
 network 192.168.10.0 255.255.255.192
 default-router 192.168.10.1
 dns-server 192.168.100.2
```

Pools: `EXEC_POOL`, `FINANCE_POOL`, `OPS_POOL`, `CUSTSERV_POOL`, `IT_POOL`, `HR_POOL`, `MARKETING_POOL`, `GUEST_POOL`.

> VLANs 100 and 200 are static only. DHCP on core SVIs handles requests from access VLANs without `ip helper-address` on L2 IDFs in our setup.
