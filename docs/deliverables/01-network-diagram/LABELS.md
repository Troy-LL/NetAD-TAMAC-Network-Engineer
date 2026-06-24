# Topology Labels — Copy/Paste for Packet Tracer

Use Packet Tracer **Place Note** (letter **N** on keyboard, or Note tool in the toolbar). Click near each colored box and paste the matching block below.

**Staff VLANs:** DHCP from CORE-SW · DNS `192.168.100.2`  
**Guest VLAN 90:** DHCP from Edge Router · DNS `8.8.8.8` · internal access blocked

---

## Backbone (center of diagram)

### ISP-Router
```
ISP-Router
WAN: 10.0.0.1/30
→ Edge Router Serial0/0/0
```

### Edge Router
```
Edge Router (2911)
WAN: 10.0.0.2/30 → NAT → ISP
Gi0/0 trunk → Core Fa0/8
  .200 = 192.168.200.14/28 (mgmt)
  .90  = 192.168.90.1/25 (guest GW + ACL)
```

### CORE-SW
```
CORE-SW (3560)
Mgmt: 192.168.200.1/28 (Vlan200)
Inter-VLAN routing + staff DHCP
Default route → 192.168.200.14
Po1–Po4 → IDFs 1-A, 1-B, 2-A, 2-B
```

### DNS-SVR
```
DNS-SVR
VLAN 100 · 192.168.100.2/28
GW 192.168.100.1
Core Fa0/1
```

### IDF switches (small labels on each)
```
IDF 1-A · Mgmt 192.168.200.2/28 · Po1 → Core Gi0/1–2
IDF 1-B · Mgmt 192.168.200.3/28 · Po2 → Core Fa0/2–3
IDF 2-A · Mgmt 192.168.200.4/28 · Po3 → Core Fa0/4–5
IDF 2-B · Mgmt 192.168.200.5/28 · Po4 → Core Fa0/6–7
```

---

## IDF 1-A zones

### Guest (light blue)
```
GUEST — VLAN 90
192.168.90.0/25
GW 192.168.90.1 (Edge Router)
DHCP: Edge Router GUEST_POOL
DNS: 8.8.8.8
SSID: TAMAC-Guest
AP-GUEST → IDF 1-A Fa0/17
❌ No internal VLAN access
```

### Executive / EXEC (blue)
```
EXECUTIVE — VLAN 10
192.168.10.0/26
GW 192.168.10.1
DHCP: .6 – .62
DNS: 192.168.100.2
PCs: EXEC, EXEC(1), EXEC(2)
IDF 1-A Fa0/1–3
```

### Finance / FIN (purple)
```
FINANCE — VLAN 20
192.168.20.0/26
GW 192.168.20.1
DHCP: .6 – .62
DNS: 192.168.100.2
PCs: FIN, FIN(1), FIN(2)
IDF 1-A Fa0/9–11
```

### Wireless EXEC (teal)
```
Wi‑Fi EXEC — VLAN 10
SSID: TAMAC-Corp-exec
AP-EXEC → IDF 1-A Fa0/18
Same GW/DNS as wired EXEC
```

### Wireless FIN (purple)
```
Wi‑Fi FIN — VLAN 20
SSID: TAMAC-Corp-fin
AP-FIN → IDF 1-A Fa0/19
STAFF-FIN → 192.168.20.x (DHCP)
DNS: 192.168.100.2
```

---

## IDF 1-B zones

### Customer Service / CS (green)
```
CUSTOMER SERVICE — VLAN 40
192.168.40.0/25
GW 192.168.40.1
DHCP: .6 – .126
DNS: 192.168.100.2
PCs: CUSTSERV, CUSTSERV(1), CUSTSERV(2)
IDF 1-B Fa0/1–3
```

### Operations / OPS (red)
```
OPERATIONS — VLAN 30
192.168.30.0/25
GW 192.168.30.1
DHCP: .6 – .126
DNS: 192.168.100.2
PCs: OPS, OPS(1), OPS(2)
IDF 1-B Fa0/15–17
```

### Wireless CS (green)
```
Wi‑Fi CS — VLAN 40
SSID: TAMAC-Corp-cs
AP-CS → IDF 1-B Fa0/21
STAFF-CS → 192.168.40.x
DNS: 192.168.100.2
```

### Wireless OPS (brown)
```
Wi‑Fi OPS — VLAN 30
SSID: TAMAC-Corp-ops
AP-OPS → IDF 1-B Fa0/22
Same GW/DNS as wired OPS
```

---

## IDF 2-A zones

### IT (pink)
```
IT — VLAN 50
192.168.50.0/26
GW 192.168.50.1
DHCP: .6 – .62
DNS: 192.168.100.2
PCs: IT, IT(1), IT(2)
IDF 2-A Fa0/1–3
```

### Human Resources / HR (dark blue)
```
HR — VLAN 70
192.168.70.0/26
GW 192.168.70.1
DHCP: .6 – .62
DNS: 192.168.100.2
PCs: HR, HR(1)
IDF 2-A Fa0/15–16
```

### Wireless IT (pink)
```
Wi‑Fi IT — VLAN 50
SSID: TAMAC-Corp-it
AP-IT → IDF 2-A Fa0/20
STAFF-IT → 192.168.50.x
DNS: 192.168.100.2
```

### Wireless HR (dark blue)
```
Wi‑Fi HR — VLAN 70
SSID: TAMAC-Corp-hr
AP-HR → IDF 2-A Fa0/21
Same GW/DNS as wired HR
```

---

## IDF 2-B zones

### Marketing / MARK (brown)
```
MARKETING — VLAN 80
192.168.80.0/26
GW 192.168.80.1
DHCP: .6 – .62
DNS: 192.168.100.2
PCs: MARK, MARK(1), MARK(2)
IDF 2-B Fa0/1–3
```

### Wireless MARK (brown)
```
Wi‑Fi MARK — VLAN 80
SSID: TAMAC-Corp-mark
AP-MARK → IDF 2-B Fa0/16
Same GW/DNS as wired MARK
```

---

## Master reference (one table)

| Zone on diagram | VLAN | Subnet | Gateway | DNS | DHCP source |
|---|---|---|---|---|---|
| Guest | 90 | 192.168.90.0/25 | 192.168.90.1 | 8.8.8.8 | Edge Router |
| Executive | 10 | 192.168.10.0/26 | 192.168.10.1 | 192.168.100.2 | CORE-SW |
| Finance | 20 | 192.168.20.0/26 | 192.168.20.1 | 192.168.100.2 | CORE-SW |
| Customer Service | 40 | 192.168.40.0/25 | 192.168.40.1 | 192.168.100.2 | CORE-SW |
| Operations | 30 | 192.168.30.0/25 | 192.168.30.1 | 192.168.100.2 | CORE-SW |
| IT | 50 | 192.168.50.0/26 | 192.168.50.1 | 192.168.100.2 | CORE-SW |
| HR | 70 | 192.168.70.0/26 | 192.168.70.1 | 192.168.100.2 | CORE-SW |
| Marketing | 80 | 192.168.80.0/26 | 192.168.80.1 | 192.168.100.2 | CORE-SW |
| DNS-SVR | 100 | 192.168.100.0/28 | 192.168.100.1 | (self) | Static |
| Infrastructure | 200 | 192.168.200.0/28 | 192.168.200.1 | — | Static |

---

## How to add labels in Packet Tracer

1. Open your `.pkt` in **Logical** view.
2. Press **N** (or click the **Note** icon in the bottom toolbar).
3. Click above or beside each colored box.
4. Paste the matching block from this file.
5. Resize the note box by dragging corners so text fits.
6. Use a **smaller font** for wireless zones (fewer lines).
7. When done: zoom to fit → screenshot → save as `logical-topology-labeled.png` in this folder.

### Labeling tips

- **One note per colored box** — matches how graders read the diagram.
- **Guest box** — always include “GW on Edge Router” and “DNS 8.8.8.8”; that’s the main difference.
- **Don’t label every PC with a fixed IP** — they use DHCP. Show the **range** (e.g. `192.168.10.6–.62`) instead.
- **DNS-SVR** is the only server with a fixed IP: `192.168.100.2`.
- Rename devices in PT (click label) to match docs: `AP-GUEST`, `STAFF-FIN`, etc.

### Quick verify after labeling

| Device | `ipconfig` should show |
|---|---|
| EXEC PC | 192.168.10.x · GW .1 · DNS .100.2 |
| Guest laptop | 192.168.90.x · GW .90.1 · DNS 8.8.8.8 |
| STAFF-FIN | 192.168.20.x · GW .20.1 · DNS .100.2 |
| IT PC | 192.168.50.x · GW .50.1 · DNS .100.2 |
