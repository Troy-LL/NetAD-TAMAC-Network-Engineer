# Session 03 — PC5 on IDF, DHCP Working

**Date:** 2025-06-24  
**Status:** Successful

## Context

After accidental **Reset Network**, configs were re-pasted. PC link was red until new PC + simulation restart; PC5 eventually worked on **IDF 1-A**.

## Placement

- **PC5** → IDF 1-A access port (Executive / VLAN 10)
- DHCP enabled on PC

## Verify on PC5 (Command Prompt)

```
ipconfig
ping 192.168.10.1
ping 192.168.100.2
ping 10.0.0.1
```

## Next

Add one PC per remaining department VLAN; then APs and security.
