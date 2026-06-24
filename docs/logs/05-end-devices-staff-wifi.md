# Session 05 — Complete End-Device Guide (PCs + Guest + Staff Wi‑Fi)

**Date:** 2025-06-24  
**Status:** Implementation guide — **applied in Packet Tracer** (logical topology + STAFF-FIN verified)

**Prerequisites:** Sessions 01–04 complete (backbone, IDFs, DHCP, guest AP, GUEST_RESTRICT ACL).

---

## Before you start

### Logical vs Physical

| View | Use for |
|---|---|
| **Logical** | IDF uplinks to Core, backbone, AP uplinks if Physical distance causes red links |
| **Physical** | Floor plan placement, PC→local IDF short cables, AP near department |

### Cable type

Everything copper: **Copper Straight-Through** (except ISP serial).

### No Core changes

Extra PCs and staff Wi‑Fi use **existing VLANs and DHCP pools**. Do not add VLANs or change `GUEST_RESTRICT` for corporate SSID.

---

## Master device table

### Wired PCs (all DHCP)

| Label | Dept | VLAN | IDF | Port | IP range |
|---|---|---|---|---|---|
| PC-EXEC | Executive | 10 | 1-A | Fa0/1 | 192.168.10.x |
| PC-EXEC-2 | Executive | 10 | 1-A | Fa0/2 | 192.168.10.x |
| PC-EXEC-3 | Executive | 10 | 1-A | Fa0/3 | 192.168.10.x |
| PC-FIN | Finance | 20 | 1-A | Fa0/9 | 192.168.20.x |
| PC-FIN-2 | Finance | 20 | 1-A | Fa0/10 | 192.168.20.x |
| PC-FIN-3 | Finance | 20 | 1-A | Fa0/11 | 192.168.20.x |
| PC-CUSTSERV | Customer Service | 40 | 1-B | Fa0/1 | 192.168.40.x |
| PC-CUSTSERV-2 | Customer Service | 40 | 1-B | Fa0/2 | 192.168.40.x |
| PC-CUSTSERV-3 | Customer Service | 40 | 1-B | Fa0/3 | 192.168.40.x |
| PC-OPS | Operations | 30 | 1-B | Fa0/15 | 192.168.30.x |
| PC-OPS-2 | Operations | 30 | 1-B | Fa0/16 | 192.168.30.x |
| PC-OPS-3 | Operations | 30 | 1-B | Fa0/17 | 192.168.30.x |
| PC-IT | IT | 50 | 2-A | Fa0/1 | 192.168.50.x |
| PC-IT-2 | IT | 50 | 2-A | Fa0/2 | 192.168.50.x |
| PC-IT-3 | IT | 50 | 2-A | Fa0/3 | 192.168.50.x |
| PC-HR | HR | 70 | 2-A | Fa0/15 | 192.168.70.x |
| PC-HR-2 | HR | 70 | 2-A | Fa0/16 | 192.168.70.x |
| PC-MARK | Marketing | 80 | 2-B | Fa0/1 | 192.168.80.x |
| PC-MARK-2 | Marketing | 80 | 2-B | Fa0/2 | 192.168.80.x |
| PC-MARK-3 | Marketing | 80 | 2-B | Fa0/3 | 192.168.80.x |

### Access points

| AP | Location | IDF | Port | VLAN | SSID | Passphrase |
|---|---|---|---|---|---|---|
| AP-Guest | Lobby F1 | 1-A | Fa0/17 | 90 | TAMAC-Guest | Guest123! |
| AP-EXEC | Executive F1 | 1-A | Fa0/18 | 10 | TAMAC-Corp-exec | TAMACstaff2024! |
| AP-FIN | Finance F1 | 1-A | Fa0/19 | 20 | TAMAC-Corp-fin | TAMACstaff2024! |
| AP-CS | Customer Service F1 | 1-B | Fa0/21 | 40 | TAMAC-Corp-cs | TAMACstaff2024! |
| AP-OPS | Operations F1 | 1-B | Fa0/22 | 30 | TAMAC-Corp-ops | TAMACstaff2024! |
| AP-IT | IT F2 | 2-A | Fa0/20 | 50 | TAMAC-Corp-it | TAMACstaff2024! |
| AP-HR | HR F2 | 2-A | Fa0/21 | 70 | TAMAC-Corp-hr | TAMACstaff2024! |
| AP-MARK | Marketing F2 | 2-B | Fa0/16 | 80 | TAMAC-Corp-mark | TAMACstaff2024! |

Cable every AP: **Port 0** → IDF port. Configure Wi‑Fi on **Port 1** (Config tab).

### Laptops

| Label | Type | Location | SSID | Expected IP |
|---|---|---|---|---|
| Laptop-Guest | Guest | Lobby F1 | TAMAC-Guest | 192.168.90.x |
| Laptop-Staff-IT | Staff | IT F2 | TAMAC-Corp-it | 192.168.50.x |
| Laptop-Staff-CS | Staff | Customer Service F1 | TAMAC-Corp-cs | 192.168.40.x |
| Laptop-Staff-FIN | Staff | Finance F1 | TAMAC-Corp-fin | 192.168.20.x |

All laptops: **Physical** → power off → **WPC300N** card → power on.

---

## Part 1 — Wired PCs

### Step 1 — Logical view

1. Click **Logical** (bottom-right).
2. Add **PC-PT** for each **-2** and **-3** device in the table (skip if already placed).
3. Rename labels for clarity.

### Step 2 — Cable

**Copper Straight-Through** → PC **FastEthernet0** → IDF port (see master table).

### Step 3 — DHCP on each PC

**Desktop** → **IP Configuration** → **DHCP**. Start simulation; wait for green link.

### Step 4 — IDF CLI (paste per switch)

#### IDF 1-A

```
enable
configure terminal
hostname IDF-1A
interface range FastEthernet0/1 - 8
 switchport mode access
 switchport access vlan 10
 no shutdown
interface FastEthernet0/18
 switchport mode access
 switchport access vlan 10
 no shutdown
interface range FastEthernet0/9 - 16
 switchport mode access
 switchport access vlan 20
 no shutdown
interface FastEthernet0/17
 switchport mode access
 switchport access vlan 90
 no shutdown
interface FastEthernet0/19
 switchport mode access
 switchport access vlan 20
 no shutdown
interface range FastEthernet0/20 - 22
 switchport mode access
 switchport access vlan 999
 shutdown
interface range FastEthernet0/23 - 24
 switchport mode trunk
 channel-group 1 mode active
interface port-channel1
 switchport mode trunk
spanning-tree mode rapid-pvst
end
```

#### IDF 1-B

```
enable
configure terminal
hostname IDF-1B
interface range FastEthernet0/1 - 14
 switchport mode access
 switchport access vlan 40
 no shutdown
interface FastEthernet0/21
 switchport mode access
 switchport access vlan 40
 no shutdown
interface range FastEthernet0/15 - 20
 switchport mode access
 switchport access vlan 30
 no shutdown
interface FastEthernet0/22
 switchport mode access
 switchport access vlan 30
 no shutdown
interface range FastEthernet0/23 - 24
 switchport mode trunk
 channel-group 2 mode active
interface port-channel2
 switchport mode trunk
spanning-tree mode rapid-pvst
end
```

#### IDF 2-A

```
enable
configure terminal
hostname IDF-2A
interface range FastEthernet0/1 - 8
 switchport mode access
 switchport access vlan 50
 no shutdown
interface FastEthernet0/20
 switchport mode access
 switchport access vlan 50
 no shutdown
interface range FastEthernet0/15 - 19
 switchport mode access
 switchport access vlan 70
 no shutdown
interface FastEthernet0/21
 switchport mode access
 switchport access vlan 70
 no shutdown
interface range FastEthernet0/9 - 14
 switchport mode access
 switchport access vlan 999
 shutdown
interface FastEthernet0/22
 switchport mode access
 switchport access vlan 999
 shutdown
interface range FastEthernet0/23 - 24
 switchport mode trunk
 channel-group 3 mode active
interface port-channel3
 switchport mode trunk
spanning-tree mode rapid-pvst
end
```

#### IDF 2-B

```
enable
configure terminal
hostname IDF-2B
interface range FastEthernet0/1 - 12
 switchport mode access
 switchport access vlan 80
 no shutdown
interface FastEthernet0/16
 switchport mode access
 switchport access vlan 80
 no shutdown
interface range FastEthernet0/13 - 15
 switchport mode access
 switchport access vlan 70
 no shutdown
interface FastEthernet0/17
 switchport mode access
 switchport access vlan 70
 no shutdown
interface range FastEthernet0/18 - 22
 switchport mode access
 switchport access vlan 999
 shutdown
interface range FastEthernet0/23 - 24
 switchport mode trunk
 channel-group 4 mode active
interface port-channel4
 switchport mode trunk
spanning-tree mode rapid-pvst
end
```

### Step 5 — Verify wired

```
show vlan brief
show interfaces status
```

On **PC-CUSTSERV-2**:

```
ipconfig
ping 192.168.40.1
ping 192.168.100.2
ping 192.168.50.1
```

---

## Part 2 — Guest Wi‑Fi (verify if done in Session 04)

### Cable

AP-Guest **Port 0** → IDF 1-A **Fa0/17**

### AP Config → Port 1

| Setting | Value |
|---|---|
| SSID | TAMAC-Guest |
| Security | WPA2-PSK |
| Passphrase | Guest123! |

### Guest laptop

Connect to **TAMAC-Guest**. Test:

```
ipconfig
ping 192.168.90.1
ping 10.0.0.1
ping 192.168.10.1
```

Last ping must **fail** (GUEST_RESTRICT).

---

## Part 3 — Corporate staff Wi‑Fi (all departments)

### Step 1 — Place APs

One **AP-PT** per department zone on the floor plan (or Logical canvas).

### Step 2 — Cable Port 0

| AP | IDF | Port |
|---|---|---|
| AP-EXEC | 1-A | Fa0/18 |
| AP-FIN | 1-A | Fa0/19 |
| AP-CS | 1-B | Fa0/21 |
| AP-OPS | 1-B | Fa0/22 |
| AP-IT | 2-A | Fa0/20 |
| AP-HR | 2-A | Fa0/21 |
| AP-MARK | 2-B | Fa0/16 |

### Step 3 — Configure Port 1 on every corporate AP

Each AP uses its **own department SSID** (see AP table above). Other settings are identical.

| Setting | Value |
|---|---|
| SSID | `TAMAC-Corp-<dept>` (e.g. `TAMAC-Corp-it`) |
| Security | WPA2-PSK |
| Passphrase | TAMACstaff2024! |
| Encryption | AES |
| Coverage Range | 150–200 |

### Step 4 — Staff laptops

1. Place laptops in IT, Customer Service, Finance.
2. Install **WPC300N**.
3. **PC Wireless** → join the matching department SSID (e.g. IT laptop → **TAMAC-Corp-it**) → `TAMACstaff2024!`

### Step 5 — Verify staff Wi‑Fi

**Laptop-Staff-FIN:**

```
ipconfig
ping 192.168.100.2
ping 192.168.10.1
ping 10.0.0.1
```

All should succeed (unlike guest).

---

## Part 4 — Physical floor plan (optional)

### Floor 1

| Zone | Devices |
|---|---|
| Executive | PC-EXEC×3, AP-EXEC |
| Finance | PC-FIN×3, AP-FIN, Laptop-Staff-FIN |
| Customer Service | PC-CUSTSERV×3, AP-CS, Laptop-Staff-CS |
| Operations | PC-OPS×3, AP-OPS |
| Lobby | AP-Guest, Laptop-Guest |
| Utility 1-A closet | IDF 1-A |
| Utility 1-B closet | IDF 1-B |

### Floor 2

| Zone | Devices |
|---|---|
| IT Server Room (MDF) | Core, Edge Router, DNS-SVR only |
| IT | PC-IT×3, AP-IT, Laptop-Staff-IT |
| HR | PC-HR×2, AP-HR |
| Marketing | PC-MARK×3, AP-MARK |
| Utility 2-A closet | IDF 2-A |
| Utility 2-B closet | IDF 2-B |

### Physical cabling rules

- PCs → **local IDF** (short cables)
- IDF uplinks → Core in **Logical** if cross-floor links go red
- AP → local IDF **Port 0**

---

## Part 5 — Full verification checklist

| # | Test | Device | Command | Expected |
|---|---|---|---|---|
| 1 | Wired DHCP | PC-FIN-2 | ipconfig | 192.168.20.x |
| 2 | Same VLAN | PC-IT + PC-IT-2 | ping each other | OK |
| 3 | Inter-VLAN | PC-IT | ping 192.168.10.1 | OK |
| 4 | DNS | PC-EXEC | ping 192.168.100.2 | OK |
| 5 | Internet | PC-OPS | ping 10.0.0.1 | OK |
| 6 | Staff Wi‑Fi DHCP | Laptop-Staff-IT | ipconfig | 192.168.50.x |
| 7 | Staff inter-VLAN | Laptop-Staff-IT | ping 192.168.20.1 | OK |
| 8 | Guest DHCP | Laptop-Guest | ipconfig | 192.168.90.x |
| 9 | Guest isolation | Laptop-Guest | ping 192.168.10.1 | Fail |
| 10 | Guest internet | Laptop-Guest | ping 10.0.0.1 | OK |
| 11 | Finance Wi‑Fi | Laptop-Staff-FIN | ipconfig | 192.168.20.x |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| No DHCP | Check IDF port VLAN; simulation on; Core DHCP pool exists |
| Red PC link Physical | Move PC closer or cable in Logical |
| Red AP uplink Physical | Cable AP→IDF in Logical |
| Wrong Wi‑Fi subnet | Connected to wrong SSID or AP on wrong VLAN |
| Guest blocked from gateway | GUEST_RESTRICT missing UDP bootps/bootpc permits |
| Staff blocked internal | Accidentally on TAMAC-Guest |

---

## After completion

- **File → Save** your `.pkt`
- Screenshot verification tests
- Next: SSH, port security, DHCP snooping → [security.md](../design/security.md)

**Final counts:** ~20 wired PCs · 8 APs · 4 laptops
