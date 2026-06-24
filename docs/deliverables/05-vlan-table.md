# TAMAC Inc. VLAN tables

These tables document the VLANs used in the implemented Packet Tracer network.

No VLAN 60 is used. TAMAC has no Procurement department. Unused ports are placed in VLAN 999 and shut down.

## VLAN summary

| VLAN ID | Name | Department or use | Subnet | Gateway | DHCP server | DNS | Routed by | Wireless SSID |
|---:|---|---|---|---|---|---|---|---|
| 10 | `EXEC` | Executive | `192.168.10.0/26` | `192.168.10.1` | `CORE-SW` | `192.168.100.2` | `CORE-SW Vlan10` | `TAMAC-Corp-exec` |
| 20 | `FINANCE` | Finance | `192.168.20.0/26` | `192.168.20.1` | `CORE-SW` | `192.168.100.2` | `CORE-SW Vlan20` | `TAMAC-Corp-fin` |
| 30 | `OPS` | Operations | `192.168.30.0/25` | `192.168.30.1` | `CORE-SW` | `192.168.100.2` | `CORE-SW Vlan30` | `TAMAC-Corp-ops` |
| 40 | `CUSTSERV` | Customer Service | `192.168.40.0/25` | `192.168.40.1` | `CORE-SW` | `192.168.100.2` | `CORE-SW Vlan40` | `TAMAC-Corp-cs` |
| 50 | `IT` | IT | `192.168.50.0/26` | `192.168.50.1` | `CORE-SW` | `192.168.100.2` | `CORE-SW Vlan50` | `TAMAC-Corp-it` |
| 70 | `HR` | Human Resources | `192.168.70.0/26` | `192.168.70.1` | `CORE-SW` | `192.168.100.2` | `CORE-SW Vlan70` | `TAMAC-Corp-hr` |
| 80 | `MARKETING` | Marketing | `192.168.80.0/26` | `192.168.80.1` | `CORE-SW` | `192.168.100.2` | `CORE-SW Vlan80` | `TAMAC-Corp-mark` |
| 90 | `GUEST` | Guest / visitors | `192.168.90.0/25` | `192.168.90.1` | `Edge-Router` | `8.8.8.8` | `Edge-Router Gi0/0.90` | `TAMAC-Guest` |
| 100 | `SERVERS` | DNS and internal services | `192.168.100.0/28` | `192.168.100.1` | Static | `192.168.100.2` | `CORE-SW Vlan100` | None |
| 200 | `MGMT` | Network device management | `192.168.200.0/28` | `192.168.200.1` | Static | None | `CORE-SW Vlan200` | None |
| 999 | `UNUSED` | Shutdown unused ports | None | None | None | None | None | None |

## Core switch VLAN port map

| Core port | VLAN or trunk | Connected device | Purpose |
|---|---|---|---|
| `Fa0/1` | Access VLAN 100 | `DNS-SVR` | DNS server connection |
| `Fa0/2` | Trunk, Po2 | `IDF-1B Fa0/23` | IDF-1B uplink |
| `Fa0/3` | Trunk, Po2 | `IDF-1B Fa0/24` | IDF-1B uplink |
| `Fa0/4` | Trunk, Po3 | `IDF-2A Fa0/23` | IDF-2A uplink |
| `Fa0/5` | Trunk, Po3 | `IDF-2A Fa0/24` | IDF-2A uplink |
| `Fa0/6` | Trunk, Po4 | `IDF-2B Fa0/23` | IDF-2B uplink |
| `Fa0/7` | Trunk, Po4 | `IDF-2B Fa0/24` | IDF-2B uplink |
| `Fa0/8` | Trunk VLANs 90, 200 | `Edge-Router Gi0/0` | Guest and management router trunk |
| `Fa0/9-24` | Access VLAN 999 | None | Shutdown unused ports |
| `Gi0/1` | Trunk, Po1 | `IDF-1A Fa0/23` | IDF-1A uplink |
| `Gi0/2` | Trunk, Po1 | `IDF-1A Fa0/24` | IDF-1A uplink |

## IDF access port VLAN map

### IDF-1A

| Port or range | VLAN | Use |
|---|---:|---|
| `Fa0/1-8` | 10 | Executive wired PCs |
| `Fa0/9-16` | 20 | Finance wired PCs |
| `Fa0/17` | 90 | Guest AP, `TAMAC-Guest` |
| `Fa0/18` | 10 | Executive AP, `TAMAC-Corp-exec` |
| `Fa0/19` | 20 | Finance AP, `TAMAC-Corp-fin` |
| `Fa0/20-22` | 999 | Shutdown unused ports |
| `Fa0/23-24` | Trunk, Po1 | Uplink to core |

### IDF-1B

| Port or range | VLAN | Use |
|---|---:|---|
| `Fa0/1-14` | 40 | Customer Service wired PCs |
| `Fa0/15-20` | 30 | Operations wired PCs |
| `Fa0/21` | 40 | Customer Service AP, `TAMAC-Corp-cs` |
| `Fa0/22` | 30 | Operations AP, `TAMAC-Corp-ops` |
| `Fa0/23-24` | Trunk, Po2 | Uplink to core |

### IDF-2A

| Port or range | VLAN | Use |
|---|---:|---|
| `Fa0/1-8` | 50 | IT wired PCs |
| `Fa0/9-14` | 999 | Shutdown unused ports, no Procurement |
| `Fa0/15-19` | 70 | HR wired PCs |
| `Fa0/20` | 50 | IT AP, `TAMAC-Corp-it` |
| `Fa0/21` | 70 | HR AP, `TAMAC-Corp-hr` |
| `Fa0/22` | 999 | Shutdown unused port |
| `Fa0/23-24` | Trunk, Po3 | Uplink to core |

### IDF-2B

| Port or range | VLAN | Use |
|---|---:|---|
| `Fa0/1-12` | 80 | Marketing wired PCs |
| `Fa0/13-15` | 70 | HR training ports |
| `Fa0/16` | 80 | Marketing AP, `TAMAC-Corp-mark` |
| `Fa0/17` | 70 | HR training / AP expansion |
| `Fa0/18-22` | 999 | Shutdown unused ports |
| `Fa0/23-24` | Trunk, Po4 | Uplink to core |

## Representative end-device VLAN map

| Device label | VLAN | Department | IDF | Port | Addressing |
|---|---:|---|---|---|---|
| `PC-EXEC` | 10 | Executive | IDF-1A | `Fa0/1` | DHCP |
| `PC-EXEC-2` | 10 | Executive | IDF-1A | `Fa0/2` | DHCP |
| `PC-EXEC-3` | 10 | Executive | IDF-1A | `Fa0/3` | DHCP |
| `PC-FIN` | 20 | Finance | IDF-1A | `Fa0/9` | DHCP |
| `PC-FIN-2` | 20 | Finance | IDF-1A | `Fa0/10` | DHCP |
| `PC-FIN-3` | 20 | Finance | IDF-1A | `Fa0/11` | DHCP |
| `PC-OPS` | 30 | Operations | IDF-1B | `Fa0/15` | DHCP |
| `PC-OPS-2` | 30 | Operations | IDF-1B | `Fa0/16` | DHCP |
| `PC-OPS-3` | 30 | Operations | IDF-1B | `Fa0/17` | DHCP |
| `PC-CUSTSERV` | 40 | Customer Service | IDF-1B | `Fa0/1` | DHCP |
| `PC-CUSTSERV-2` | 40 | Customer Service | IDF-1B | `Fa0/2` | DHCP |
| `PC-CUSTSERV-3` | 40 | Customer Service | IDF-1B | `Fa0/3` | DHCP |
| `PC-IT` | 50 | IT | IDF-2A | `Fa0/1` | DHCP |
| `PC-IT-2` | 50 | IT | IDF-2A | `Fa0/2` | DHCP |
| `PC-IT-3` | 50 | IT | IDF-2A | `Fa0/3` | DHCP |
| `PC-HR` | 70 | HR | IDF-2A | `Fa0/15` | DHCP |
| `PC-HR-2` | 70 | HR | IDF-2A | `Fa0/16` | DHCP |
| `PC-MARK` | 80 | Marketing | IDF-2B | `Fa0/1` | DHCP |
| `PC-MARK-2` | 80 | Marketing | IDF-2B | `Fa0/2` | DHCP |
| `PC-MARK-3` | 80 | Marketing | IDF-2B | `Fa0/3` | DHCP |
| `Guest laptop` | 90 | Guest | Wireless via AP-GUEST | IDF-1A `Fa0/17` | DHCP from Edge Router |

## Wireless VLAN map

| AP | VLAN | SSID | IDF | Switch port | Expected client IP |
|---|---:|---|---|---|---|
| `AP-GUEST` | 90 | `TAMAC-Guest` | IDF-1A | `Fa0/17` | `192.168.90.x` |
| `AP-EXEC` | 10 | `TAMAC-Corp-exec` | IDF-1A | `Fa0/18` | `192.168.10.x` |
| `AP-FIN` | 20 | `TAMAC-Corp-fin` | IDF-1A | `Fa0/19` | `192.168.20.x` |
| `AP-OPS` | 30 | `TAMAC-Corp-ops` | IDF-1B | `Fa0/22` | `192.168.30.x` |
| `AP-CS` | 40 | `TAMAC-Corp-cs` | IDF-1B | `Fa0/21` | `192.168.40.x` |
| `AP-IT` | 50 | `TAMAC-Corp-it` | IDF-2A | `Fa0/20` | `192.168.50.x` |
| `AP-HR` | 70 | `TAMAC-Corp-hr` | IDF-2A | `Fa0/21` | `192.168.70.x` |
| `AP-MARK` | 80 | `TAMAC-Corp-mark` | IDF-2B | `Fa0/16` | `192.168.80.x` |

## Trunk and EtherChannel summary

| EtherChannel | Core ports | IDF ports | Carries VLANs | Purpose |
|---|---|---|---|---|
| Po1 | `Gi0/1-2` | IDF-1A `Fa0/23-24` | Staff, guest, server, mgmt VLANs | IDF-1A uplink |
| Po2 | `Fa0/2-3` | IDF-1B `Fa0/23-24` | Staff, guest, server, mgmt VLANs | IDF-1B uplink |
| Po3 | `Fa0/4-5` | IDF-2A `Fa0/23-24` | Staff, guest, server, mgmt VLANs | IDF-2A uplink |
| Po4 | `Fa0/6-7` | IDF-2B `Fa0/23-24` | Staff, guest, server, mgmt VLANs | IDF-2B uplink |
| Router trunk | `CORE-SW Fa0/8` | `Edge-Router Gi0/0` | VLANs 90, 200 | Guest gateway and router management |

## Notes for reviewers

- Guest traffic uses VLAN 90, but the gateway and ACL are on `Edge-Router Gi0/0.90`.
- Corporate Wi-Fi uses separate SSIDs per department. This is intentional so staff devices join the correct VLAN.
- VLAN 999 has no gateway and is only for shutdown unused ports.
- IDF-2A `Fa0/9-14` stay shutdown on VLAN 999 because there is no Procurement department.
