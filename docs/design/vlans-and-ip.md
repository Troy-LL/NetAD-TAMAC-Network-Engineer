# VLANs & IP Addressing

[← Source of truth](../../cisco.mdc)

**Base:** `192.168.0.0/16` · **Routing:** Static · **DNS for all pools:** `192.168.100.2`

## VLAN table

| VLAN | Name | Purpose |
|---|---|---|
| 10 | EXEC | Executive |
| 20 | FINANCE | Finance |
| 30 | OPS | Operations |
| 40 | CUSTSERV | Customer Service |
| 50 | IT | IT |
| 70 | HR | Human Resources |
| 80 | MARKETING | Marketing |
| 90 | GUEST | Guest / visitors |
| 100 | SERVERS | DNS & internal services |
| 200 | MGMT | Device management |
| 999 | UNUSED | Shutdown unused ports |

No VLAN 60.

## Subnets & gateways

| Zone | Network | Mask | CIDR | Gateway |
|---|---|---|---|---|
| Executive | 192.168.10.0 | 255.255.255.192 | /26 | 192.168.10.1 |
| Finance | 192.168.20.0 | 255.255.255.192 | /26 | 192.168.20.1 |
| Operations | 192.168.30.0 | 255.255.255.128 | /25 | 192.168.30.1 |
| Customer Service | 192.168.40.0 | 255.255.255.128 | /25 | 192.168.40.1 |
| IT | 192.168.50.0 | 255.255.255.192 | /26 | 192.168.50.1 |
| HR | 192.168.70.0 | 255.255.255.192 | /26 | 192.168.70.1 |
| Marketing | 192.168.80.0 | 255.255.255.192 | /26 | 192.168.80.1 |
| Guest | 192.168.90.0 | 255.255.255.128 | /25 | **192.168.90.1** (Edge Router) |
| Servers | 192.168.100.0 | 255.255.255.240 | /28 | 192.168.100.1 |
| Management | 192.168.200.0 | 255.255.255.240 | /28 | 192.168.200.1 |
| WAN (ISP) | 10.0.0.0 | 255.255.255.252 | /30 | 10.0.0.1 (ISP) |

## Static assignments

| Device | IP | VLAN |
|---|---|---|
| Core switch (SVI) | 192.168.200.1 | 200 |
| Edge router LAN | **192.168.200.14** | 200 |
| Edge router WAN | 10.0.0.2 | — |
| DNS-SVR | 192.168.100.2 | 100 |
| Edge router guest GW | **192.168.90.1** | 90 (Gi0/0.90) |
| IDF 1-A mgmt | 192.168.200.2 | 200 |
| IDF 1-B mgmt | 192.168.200.3 | 200 |
| IDF 2-A mgmt | 192.168.200.4 | 200 |
| IDF 2-B mgmt | 192.168.200.5 | 200 |
