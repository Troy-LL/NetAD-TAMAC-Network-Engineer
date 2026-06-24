# Guest Network

[← Source of truth](../../cisco.mdc)

| Setting | Value |
|---|---|
| VLAN | 90 |
| Subnet | 192.168.90.0/25 |
| SSID | TAMAC-Guest |
| Auth | WPA2-PSK (`Guest123!`) |
| Gateway | **192.168.90.1** on Edge Router Gi0/0.90 |
| DHCP | GUEST_POOL on **Edge Router** Gi0/0.90 |
| DNS | 8.8.8.8 (guest pool) |
| Internet | Via NAT on router |
| Internal access | Blocked by GUEST_RESTRICT ACL on Gi0/0.90 |

> Core `Vlan90` SVI is **shutdown** — guest L3 terminates on the edge router.

## AP (IDF 1-A Fa0/17)

- SSID: `TAMAC-Guest`
- Access port VLAN 90 (not trunk)

## Traffic flow

```
Guest device → AP (VLAN 90) → IDF 1-A → Core trunk → Edge Router Gi0/0.90
  → GUEST_RESTRICT blocks internal 192.168.0.0/16
  → Permitted traffic → NAT → ISP
```

ACL config: [security.md](security.md) · Router subinterface fix: [log 07](../logs/07-guest-acl-router-fix.md)
