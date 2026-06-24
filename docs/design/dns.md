# DNS Server

[← Source of truth](../../cisco.mdc)

| Field | Value |
|---|---|
| Device | Server-PT |
| Hostname | DNS-SVR |
| IP | 192.168.100.2 / 255.255.255.240 |
| Gateway | 192.168.100.1 |
| VLAN | 100 |
| Cable | Core **Fa0/1** (access) |

## A records (Config → SERVICES → DNS)

| Name | Address |
|---|---|
| tamac.local | 192.168.100.2 |
| dns.tamac.local | 192.168.100.2 |
| www.tamac.local | 192.168.100.2 |
| gateway.tamac.local | **192.168.200.14** |

Turn DNS service **ON**.

## Test

From core: `ping 192.168.100.2` (requires SERVER_ACCESS with same-subnet permit — see [security.md](security.md)).
