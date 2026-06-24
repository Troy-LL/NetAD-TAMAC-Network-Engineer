# Ports & Cabling

[← Source of truth](../../cisco.mdc)

## Cable types

| Link | Cable |
|---|---|
| ISP ↔ Router | Serial DCE |
| Everything else (copper) | Copper Straight-Through |

Router needs **HWIC-2T** module for Serial0/0/0.

## Edge router

| Port | To | IP |
|---|---|---|
| Serial0/0/0 | ISP Cloud | 10.0.0.2 /30 |
| GigabitEthernet0/0 | Core **Fa0/8** | 192.168.200.14 /28 |

## Core switch (3560-24PS)

| Port | To | Type |
|---|---|---|
| Fa0/8 | Router Gi0/0 | Access VLAN 200 |
| Fa0/1 | DNS-SVR | Access VLAN 100 |
| Gi0/1 | IDF 1-A Fa0/23 | Trunk Po1 |
| Gi0/2 | IDF 1-A Fa0/24 | Trunk Po1 |
| Fa0/2 | IDF 1-B Fa0/23 | Trunk Po2 |
| Fa0/3 | IDF 1-B Fa0/24 | Trunk Po2 |
| Fa0/4 | IDF 2-A Fa0/23 | Trunk Po3 |
| Fa0/5 | IDF 2-A Fa0/24 | Trunk Po3 |
| Fa0/6 | IDF 2-B Fa0/23 | Trunk Po4 |
| Fa0/7 | IDF 2-B Fa0/24 | Trunk Po4 |
| Fa0/9–24 | — | Shutdown VLAN 999 |

## IDF 1-A

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–8 | 10 | Executive PCs |
| Fa0/9–16 | 20 | Finance PCs |
| Fa0/17 | 90 | Guest AP |
| Fa0/18 | 10 | Executive AP |
| Fa0/19 | 20 | Finance AP |
| Fa0/20–22 | 999 | Shutdown |
| Fa0/23–24 | Trunk | Po1 → Core |

## IDF 1-B

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–14, Fa0/21 | 40 | Customer Service |
| Fa0/15–20, Fa0/22 | 30 | Operations |
| Fa0/23–24 | Trunk | Po2 → Core |

## IDF 2-A

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–8, Fa0/20 | 50 | IT |
| Fa0/9–14 | 999 | Shutdown (unused) |
| Fa0/15–19, Fa0/21 | 70 | HR |
| Fa0/22 | 999 | Shutdown |
| Fa0/23–24 | Trunk | Po3 → Core |

## IDF 2-B

| Ports | VLAN | Use |
|---|---|---|
| Fa0/1–12, Fa0/16 | 80 | Marketing |
| Fa0/13–15, Fa0/17 | 70 | HR Training |
| Fa0/18–22 | 999 | Shutdown |
| Fa0/23–24 | Trunk | Po4 → Core |

## End devices (quick)

Full table: [end-devices.md](end-devices.md) · Staff Wi‑Fi: [corporate-wifi.md](corporate-wifi.md)

| Type | Cable | Notes |
|---|---|---|
| PC-PT | Fa0 → IDF access port | DHCP |
| AP-PT | **Port 0** → IDF access port | Wi‑Fi on **Port 1** (Config) |
| Staff laptop | Wireless → `TAMAC-Corp-<dept>` | WPC300N module required |
| Guest laptop | Wireless → `TAMAC-Guest` | VLAN 90 only |
