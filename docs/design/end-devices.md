# End Devices — PCs, Laptops, Access Points

[← Source of truth](../../cisco.mdc)

TAMAC represents **200–250 employees** with **subnet sizing and port maps**, not one PC per person. Packet Tracer uses **representative wired PCs**, **per-department corporate APs** (`TAMAC-Corp-<dept>`), and **guest Wi‑Fi** (`TAMAC-Guest`).

## Scale model

| Layer | How 200+ staff is represented |
|---|---|
| Subnets | `/25`–`/26` per department (30–126 hosts) |
| Switch ports | 8–14 access ports per VLAN on IDFs |
| Corporate Wi‑Fi | One AP per zone → many laptops |
| Packet Tracer | 2–3 wired PCs per dept + 1–2 staff laptops |

## Wired PC port map

Cable **Copper Straight-Through** → PC `FastEthernet0` → IDF port. All PCs use **DHCP**.

| Label | VLAN | IDF | Port |
|---|---|---|---|
| PC-EXEC | 10 | 1-A | Fa0/1 |
| PC-EXEC-2 | 10 | 1-A | Fa0/2 |
| PC-EXEC-3 | 10 | 1-A | Fa0/3 |
| PC-FIN | 20 | 1-A | Fa0/9 |
| PC-FIN-2 | 20 | 1-A | Fa0/10 |
| PC-FIN-3 | 20 | 1-A | Fa0/11 |
| PC-CUSTSERV | 40 | 1-B | Fa0/1 |
| PC-CUSTSERV-2 | 40 | 1-B | Fa0/2 |
| PC-CUSTSERV-3 | 40 | 1-B | Fa0/3 |
| PC-OPS | 30 | 1-B | Fa0/15 |
| PC-OPS-2 | 30 | 1-B | Fa0/16 |
| PC-OPS-3 | 30 | 1-B | Fa0/17 |
| PC-IT | 50 | 2-A | Fa0/1 |
| PC-IT-2 | 50 | 2-A | Fa0/2 |
| PC-IT-3 | 50 | 2-A | Fa0/3 |
| PC-HR | 70 | 2-A | Fa0/15 |
| PC-HR-2 | 70 | 2-A | Fa0/16 |
| PC-MARK | 80 | 2-B | Fa0/1 |
| PC-MARK-2 | 80 | 2-B | Fa0/2 |
| PC-MARK-3 | 80 | 2-B | Fa0/3 |

> If a PC name already exists (e.g. `PC-PT EXEC`), keep it on Fa0/1 and add `-2`/`-3` PCs on the next ports.

### Hacker PCs (port-security demo only)

Add one **unconnected** PC-PT per department you want to demo, labeled **`HACK-<dept>`** (e.g. `HACK-EXEC`, `HACK-IT`). **Do not cable them to the switch.** During the defense demo, unplug a legitimate PC’s cable and attach it to the hacker PC — port security should **err-disable** the switch port (red link). Reconnect the cable to the original PC and bounce the port (`shutdown` / `no shutdown`) to restore the link.

Full steps: [log 11](../logs/11-port-security-hacker-pc-demo.md).

## Wireless summary

| SSID | VLAN | Users | Doc |
|---|---|---|---|
| `TAMAC-Guest` | 90 | Visitors | [guest-network.md](guest-network.md) |
| `TAMAC-Corp-<dept>` | Per dept (10/20/30/40/50/70/80) | Staff laptops | [corporate-wifi.md](corporate-wifi.md) |

## Implementation guide

Full step-by-step + CLI: [docs/logs/05-end-devices-staff-wifi.md](../logs/05-end-devices-staff-wifi.md)
