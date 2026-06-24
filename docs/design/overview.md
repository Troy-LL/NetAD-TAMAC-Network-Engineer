# Overview — Company, Topology, Hardware

[← Source of truth](../../cisco.mdc)

## Company

| Field | Value |
|---|---|
| Name | TAMAC Inc. |
| Business | Centralized e-commerce platform aggregator |
| Employees | 200–250 |
| Building | Two-story HQ |
| Option | Option 4 — E-Commerce Company |

**Departments (8):** Executive, IT, Operations, Finance, Customer Service, Marketing, HR, Guest.  
**No Procurement department.**

## Layer structure

```
[ ISP / Internet Cloud ]
         |
  [ Edge Router — 2911 ]     NAT/PAT, default route
         |
[ Core Switch — 3560 ]       SVIs, DHCP, ACLs, static routes
    /    |    \    \
  IDF  IDF   IDF  IDF
  1-A  1-B   2-A  2-B
```

## Floor plan (brief)

**Floor 1:** Lobby/Guest · Executive · Finance · Operations · Customer Service · Utility 1-A, 1-B  
**Floor 2:** IT + MDF · HR · Marketing · Utility 2-A, 2-B  
**MDF (IT Server Room):** Edge Router, Core Switch, DNS-SVR

## Hardware (Packet Tracer)

| Role | Model | Qty | Location |
|---|---|---|---|
| ISP | Cloud-PT | 1 | External |
| Edge router | Cisco 2911 (+ HWIC-2T for serial) | 1 | MDF |
| Core | Cisco 3560-24PS | 1 | MDF |
| Access | Cisco 2960-24TT | 4 | IDF 1-A, 1-B, 2-A, 2-B |
| DNS | Server-PT | 1 | MDF |
| APs | AP-PT | 7–8 | Per zone incl. Finance — [corporate-wifi.md](corporate-wifi.md) |
| PCs | PC-PT | 2–3 per dept | IDF access ports — [end-devices.md](end-devices.md) |
| Staff laptops | Laptop-PT | 1–2 demo | `TAMAC-Corp-<dept>` — [corporate-wifi.md](corporate-wifi.md) |
| Guest laptops | Laptop-PT | 1–2 | Lobby · `TAMAC-Guest` |

## AP placement

| AP | Location | SSID | VLAN |
|---|---|---|---|
| AP-1 | Lobby | TAMAC-Guest | 90 |
| AP-2 | Executive | TAMAC-Corp-exec | 10 |
| AP-FIN | Finance | TAMAC-Corp-fin | 20 |
| AP-3 | Customer Service | TAMAC-Corp-cs | 40 |
| AP-4 | Operations | TAMAC-Corp-ops | 30 |
| AP-5 | IT | TAMAC-Corp-it | 50 |
| AP-6 | HR | TAMAC-Corp-hr | 70 |
| AP-7 | Marketing | TAMAC-Corp-mark | 80 |
