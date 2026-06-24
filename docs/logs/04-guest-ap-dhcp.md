# Session 04 — Guest AP, DHCP, ACL Fix

**Date:** 2025-06-24  
**Status:** Successful

## Guest setup

- AP on IDF 1-A Fa0/17 (VLAN 90), SSID `TAMAC-Guest`, WPA2-PSK
- Laptop wireless connected

## GUEST_RESTRICT ACL fix (CORE-SW)

Original `deny ip 192.168.90.0 … 192.168.0.0 0.0.255.255` blocked gateway + DHCP.

Final ACL must include **DHCP permits first**:

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

## Verified (Guest laptop)

- DHCP: working
- ping 192.168.90.1: OK
- ping 10.0.0.1: OK
- ping 192.168.10.1: blocked (isolation)
