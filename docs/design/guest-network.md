# Guest Network

[← Source of truth](../../cisco.mdc)

| Setting | Value |
|---|---|
| VLAN | 90 |
| Subnet | 192.168.90.0/25 |
| SSID | TAMAC-Guest |
| Auth | WPA2-PSK |
| DHCP | GUEST_POOL on core |
| Internet | Via NAT on router |
| Internal access | Blocked by GUEST_RESTRICT ACL |

## AP (IDF 1-A Fa0/17)

- SSID: `TAMAC-Guest`
- Access port VLAN 90 (not trunk)

## Traffic flow

```
Guest device → AP (VLAN 90) → IDF 1-A → Core
  → GUEST_RESTRICT blocks 192.168.0.0/16
  → Default route → Router → NAT → ISP
```

ACL config: [security.md](security.md)
