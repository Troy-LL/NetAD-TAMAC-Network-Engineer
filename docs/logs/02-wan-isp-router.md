# Session 02 — WAN via ISP-Router

**Date:** 2025-06-24  
**Status:** Successful

## Change

Cloud-PT had no clock rate field → serial stayed `up/down`. Replaced with **ISP-Router (2911)**.

## ISP-Router config

```
hostname ISP-Router
interface Serial0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
ip route 192.168.0.0 255.255.0.0 10.0.0.2
```

Cable: Serial DCE — ISP-Router Serial0/0/0 → Edge Router Serial0/0/0

## Edge Router

```
ip route 0.0.0.0 0.0.0.0 10.0.0.1
! NAT already configured: acl 1, nat inside Gi0/0, nat outside Serial0/0/0, overload
```

## Verified output

```
Router# ping 10.0.0.1
!!!!!  Success rate is 100 percent (5/5)

Router# show ip route
Gateway of last resort is 10.0.0.1 to network 0.0.0.0
S*   0.0.0.0/0 [1/0] via 10.0.0.1
```

## Next

Add PCs → DHCP → ping 192.168.100.2 and 10.0.0.1 from PC
