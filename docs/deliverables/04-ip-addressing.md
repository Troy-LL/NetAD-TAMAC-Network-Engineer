# TAMAC Inc. IP addressing tables

These tables document the implemented Packet Tracer addressing plan. Staff VLANs use DHCP from `CORE-SW`. Guest VLAN 90 uses DHCP from `Edge-Router`.

No VLAN 60 is used. TAMAC has no Procurement department.

## Addressing summary

| Area | Addressing method | DHCP server | DNS |
|---|---|---|---|
| Staff VLANs | DHCP | `CORE-SW` | `192.168.100.2` |
| Guest VLAN | DHCP | `Edge-Router` | `8.8.8.8` |
| Servers | Static | None | `192.168.100.2` |
| Management | Static | None | None |
| WAN | Static | None | None |

## Department DHCP networks

| Department | VLAN | Network | Mask | CIDR | Default gateway | DHCP range | DNS |
|---|---:|---|---|---:|---|---|---|
| Executive | 10 | `192.168.10.0` | `255.255.255.192` | `/26` | `192.168.10.1` | `192.168.10.6` - `192.168.10.62` | `192.168.100.2` |
| Finance | 20 | `192.168.20.0` | `255.255.255.192` | `/26` | `192.168.20.1` | `192.168.20.6` - `192.168.20.62` | `192.168.100.2` |
| Operations | 30 | `192.168.30.0` | `255.255.255.128` | `/25` | `192.168.30.1` | `192.168.30.6` - `192.168.30.126` | `192.168.100.2` |
| Customer Service | 40 | `192.168.40.0` | `255.255.255.128` | `/25` | `192.168.40.1` | `192.168.40.6` - `192.168.40.126` | `192.168.100.2` |
| IT | 50 | `192.168.50.0` | `255.255.255.192` | `/26` | `192.168.50.1` | `192.168.50.6` - `192.168.50.62` | `192.168.100.2` |
| HR | 70 | `192.168.70.0` | `255.255.255.192` | `/26` | `192.168.70.1` | `192.168.70.6` - `192.168.70.62` | `192.168.100.2` |
| Marketing | 80 | `192.168.80.0` | `255.255.255.192` | `/26` | `192.168.80.1` | `192.168.80.6` - `192.168.80.62` | `192.168.100.2` |
| Guest | 90 | `192.168.90.0` | `255.255.255.128` | `/25` | `192.168.90.1` | `192.168.90.6` - `192.168.90.126` | `8.8.8.8` |

## Infrastructure static IPs

| Device | Interface | IP address | Mask | CIDR | VLAN | Notes |
|---|---|---|---|---:|---:|---|
| `CORE-SW` | `Vlan10` | `192.168.10.1` | `255.255.255.192` | `/26` | 10 | Executive gateway |
| `CORE-SW` | `Vlan20` | `192.168.20.1` | `255.255.255.192` | `/26` | 20 | Finance gateway |
| `CORE-SW` | `Vlan30` | `192.168.30.1` | `255.255.255.128` | `/25` | 30 | Operations gateway |
| `CORE-SW` | `Vlan40` | `192.168.40.1` | `255.255.255.128` | `/25` | 40 | Customer Service gateway |
| `CORE-SW` | `Vlan50` | `192.168.50.1` | `255.255.255.192` | `/26` | 50 | IT gateway |
| `CORE-SW` | `Vlan70` | `192.168.70.1` | `255.255.255.192` | `/26` | 70 | HR gateway |
| `CORE-SW` | `Vlan80` | `192.168.80.1` | `255.255.255.192` | `/26` | 80 | Marketing gateway |
| `CORE-SW` | `Vlan90` | None | None | None | 90 | Shutdown. Guest gateway moved to Edge Router |
| `CORE-SW` | `Vlan100` | `192.168.100.1` | `255.255.255.240` | `/28` | 100 | Server VLAN gateway |
| `CORE-SW` | `Vlan200` | `192.168.200.1` | `255.255.255.240` | `/28` | 200 | Management gateway |
| `Edge-Router` | `Gi0/0.90` | `192.168.90.1` | `255.255.255.128` | `/25` | 90 | Guest gateway, DHCP, ACL |
| `Edge-Router` | `Gi0/0.200` | `192.168.200.14` | `255.255.255.240` | `/28` | 200 | Core default route target |
| `Edge-Router` | `Serial0/0/0` | `10.0.0.2` | `255.255.255.252` | `/30` | None | WAN to ISP |
| `ISP-Router` | Serial link | `10.0.0.1` | `255.255.255.252` | `/30` | None | ISP side of WAN link |
| `DNS-SVR` | `Fa0` | `192.168.100.2` | `255.255.255.240` | `/28` | 100 | Internal DNS server |
| `IDF-1A` | `Vlan200` | `192.168.200.2` | `255.255.255.240` | `/28` | 200 | Switch management |
| `IDF-1B` | `Vlan200` | `192.168.200.3` | `255.255.255.240` | `/28` | 200 | Switch management |
| `IDF-2A` | `Vlan200` | `192.168.200.4` | `255.255.255.240` | `/28` | 200 | Switch management |
| `IDF-2B` | `Vlan200` | `192.168.200.5` | `255.255.255.240` | `/28` | 200 | Switch management |

## WAN addressing

| Link | Network | Mask | Router side | ISP side | Purpose |
|---|---|---|---|---|---|
| Edge Router to ISP-Router | `10.0.0.0/30` | `255.255.255.252` | `10.0.0.2` | `10.0.0.1` | Internet/WAN simulation |

## DNS addressing

| Client type | DNS server | Notes |
|---|---|---|
| Staff wired PCs | `192.168.100.2` | Assigned by core DHCP pools |
| Staff wireless laptops | `192.168.100.2` | Same VLAN/DHCP behavior as wired department devices |
| Guest wireless laptop | `8.8.8.8` | Assigned by Edge Router guest DHCP pool |
| DNS server itself | `192.168.100.2` | Static server address on VLAN 100 |

## Default routes

| Device | Default route | Next hop |
|---|---|---|
| `CORE-SW` | `0.0.0.0/0` | `192.168.200.14` |
| `Edge-Router` | `0.0.0.0/0` | `10.0.0.1` |

## Edge Router internal routes

The Edge Router sends internal staff and server networks back to the core switch at `192.168.200.1`.

| Network | Mask | Next hop |
|---|---|---|
| `192.168.10.0` | `255.255.255.192` | `192.168.200.1` |
| `192.168.20.0` | `255.255.255.192` | `192.168.200.1` |
| `192.168.30.0` | `255.255.255.128` | `192.168.200.1` |
| `192.168.40.0` | `255.255.255.128` | `192.168.200.1` |
| `192.168.50.0` | `255.255.255.192` | `192.168.200.1` |
| `192.168.70.0` | `255.255.255.192` | `192.168.200.1` |
| `192.168.80.0` | `255.255.255.192` | `192.168.200.1` |
| `192.168.100.0` | `255.255.255.240` | `192.168.200.1` |

## Notes for reviewers

- `192.168.200.14` is correct for the Edge Router management subinterface. `192.168.200.254` does not fit inside the `/28` management subnet.
- VLAN 90 is present on the core as a Layer 2 VLAN, but its gateway is on `Edge-Router Gi0/0.90`.
- VLANs 100 and 200 use static addressing only.
- VLAN 999 is for unused/shutdown ports and has no gateway.
