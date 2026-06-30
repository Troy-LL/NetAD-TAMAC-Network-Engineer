# Screenshot & export redo checklist

**Date:** 2025-06-29 (updated after recapture batch)  
**Why:** New PCs/laptops, AP port-security fix (log 10), guest multi-client DHCP, Po2 cable fix.

**VLANs and IP subnets did not change** — gateways, SSIDs, and pool names are the same. Most connectivity/routing/inter-VLAN proof shots are still valid if tests still pass.

---

## Do not redo (still accurate)

| Asset | Reason |
|---|---|
| `04-ip-addressing.md` | Subnet/gateway table unchanged |
| `04-ip-addressing-screenshots/01-core-sw-svi-addresses.png` | SVI IPs unchanged |
| `04-ip-addressing-screenshots/02-edge-router-interface-addresses.png` | Router subifs unchanged |
| `05-vlan-table.md` | VLAN list unchanged (add AP port-security note optional) |
| `05-vlan-table-screenshots/01-core-sw-vlan-brief.png` | VLAN IDs unchanged |
| `05-vlan-table-screenshots/03-idf1a-trunk-vlans.png` | Trunk VLANs unchanged |
| `05-vlan-table-screenshots/04-edge-router-guest-vlan90.png` | Gi0/0.90 still guest GW |
| `06-verification-testing/screenshots/01-connectivity/*` | Same VLAN 10 exec tests |
| `06-verification-testing/screenshots/02-inter-vlan/*` | Inter-VLAN routing unchanged |
| `06-verification-testing/screenshots/03-routing/*` | Routes/NAT unchanged |
| `06-verification-testing/screenshots/04-dhcp/01-core-dhcp-pools.png` | Staff pools unchanged |
| `06-verification-testing/screenshots/04-dhcp/02-core-dhcp-pools-continued.png` | Staff pools unchanged |
| `06-verification-testing/screenshots/04-dhcp/03-exec-dhcp-renew.png` | Still valid if exec DHCP works |
| `06-verification-testing/screenshots/04-dhcp/06-edge-router-guest-pool.png` | Pool definition unchanged |
| `06-verification-testing/screenshots/05-security/01–10, 05–07, 15–17` | ACL/SSH/telnet proof unchanged |
| `03-screenshots/02-connectivity/*` | Legacy exec proof — still valid |
| `03-screenshots/04-security/01-it-ssh-core-sw.png` | SSH proof unchanged |
| `03-screenshots/04-security/02-it-ssh-idf2b.png` | SSH proof unchanged |

---

## Must redo (stale vs built reality)

### ✅ Recaptured 2025-06-29 (batch 1)

| File | Status |
|---|---|
| `01-network-diagram/logical-topology-labeled.png` | ✅ Updated — per-dept SSIDs, guest laptops, VLAN notes |
| `01-network-diagram/logical-topology-unlabeled.png` | ✅ Updated — see **LOG box warning** below |
| `03-screenshots/05-ha/01-core-etherchannel-summary.png` | ✅ Updated — Po1–Po4 all SU, Fa0/2–3 both (P) |
| `06-verification-testing/screenshots/05-security/11b-idf1a-port-security-ap-fa0-17.png` | ✅ **New** — max 50, restrict, VLAN 90 |
| `06-verification-testing/screenshots/04-dhcp/08-edge-router-guest-binding.png` | ✅ Updated — guest lease proof |

### 1. Network diagram — Deliverable 1

| File | Action |
|---|---|
| `01-network-diagram/logical-topology-labeled.png` | ~~Recapture~~ **Done** |
| `01-network-diagram/logical-topology-unlabeled.png` | ~~Optional~~ **Done** — remove **LOG** zone if not in scope |

**Before screenshot:** Fix **CORE Fa0/3 ↔ IDF 1-B Fa0/24** if Po2 was missing; confirm all EtherChannels green.

---

### 2. Device configs — Deliverable 2

Re-export `show running-config` and replace CLI screenshots after saving `.pkt`.

| File / folder | What changed |
|---|---|
| `02-device-configs/idf-1a-running-config.txt` | Fa0/17 `maximum 50`, Fa0/18–19 `maximum 10`, sticky MACs on new PC ports |
| `02-device-configs/idf-1b-running-config.txt` | New PC ports + AP port-security if applied |
| `02-device-configs/idf-2a-running-config.txt` | New PC ports + Fa0/20–21 AP port-security |
| `02-device-configs/idf-2b-running-config.txt` | New PC ports + Fa0/16 AP port-security |
| `02-device-configs/idf-1a/02-running-config-access-ports.png` | More Fa0/9–16 finance ports in use |
| `02-device-configs/idf-1a/03-running-config-uplinks-mgmt-vty.png` | Include Fa0/17 port-security block |
| **New screenshot** `idf-1a/04-port-security-ap-fa0-17.png` | `show port-security interface FastEthernet0/17` → max **50**, violation **restrict** |
| Other IDF `01–03` screenshots | Recapture if access-port sections changed |

Core + edge router configs: redo only if you ran `no ip dhcp snooping` on core (runtime change).

---

### 3. High availability — legacy `03-screenshots/05-ha/`

| File | Action |
|---|---|
| `03-screenshots/05-ha/01-core-etherchannel-summary.png` | ~~Must redo~~ **Done** |

**Command:** `show etherchannel summary` on CORE-SW

---

### 4. Port security — verification phase 5 + legacy

| File | Action |
|---|---|
| `06-verification-testing/screenshots/05-security/11-idf1a-port-security-status.png` | Recapture Fa0/1 (wired max 1) **or** add second shot for AP |
| `03-screenshots/04-security/03-idf1a-port-security.png` | Same — update or split wired vs AP |
| **New** `05-security/11b-idf1a-port-security-ap-fa0-17.png` | ~~New~~ **Done** |

**Commands on IDF 1-A:**

```text
show port-security interface FastEthernet0/1
show port-security interface FastEthernet0/17
```

---

### 5. DHCP — guest bindings (multi-laptop)

| File | Action |
|---|---|
| `06-verification-testing/screenshots/04-dhcp/07-guest-laptop-dhcp.png` | Recapture — guest laptop with valid `192.168.90.x` |
| `06-verification-testing/screenshots/04-dhcp/08-edge-router-guest-binding.png` | ✅ Updated — recapture with **all guest laptops on** if you want multiple leases visible |
| `03-screenshots/03-guest/01-guest-laptop-dhcp.png` | Redo to match current guest DHCP |
| **New (optional)** `04-dhcp/09-multi-guest-dhcp-bindings.png` | `show ip dhcp binding` with multiple guest MACs |

---

### 6. Guest network — phase 6 + legacy

| File | Action |
|---|---|
| `06-verification-testing/screenshots/06-guest/01-guest-wireless-connected.png` | Redo if SSID/association view changed |
| `06-verification-testing/screenshots/06-guest/02-guest-laptop-dhcp.png` | Redo after Fa0/17 fix |
| `03-screenshots/03-guest/02-guest-isolation-blocked.png` | Redo if guest IP changed |
| `03-screenshots/03-guest/03-guest-ping-internet.png` | Redo if needed for consistency |

**Keep:** `06-guest/03–05` isolation/internet shots if tests still pass with same output.

**Optional new:** Logical view with **5 guest laptops**, AP link **green**, VLAN 90 note visible.

---

### 7. VLAN table proof (optional)

| File | Action |
|---|---|
| `05-vlan-table-screenshots/02-idf1a-interface-vlan-assignments.png` | Redo if more ports show **connected** for new PCs |

---

## DHCP snooping screenshots — add a note, don’t delete

| File | Status |
|---|---|
| `05-security/13-core-dhcp-snooping.png` | Shows **design/config** — OK if caption says “configured in running-config” |
| `05-security/15-dhcp-snooping-config.png` | Same |

**Runtime:** `no ip dhcp snooping` on core + IDFs for PT ([log 09](../../logs/09-dhcp-snooping-pt-workaround.md)). Add one line in defense: *“Disabled at runtime in Packet Tracer so DHCP works; config remains in export.”*

Optional new shot: `show ip dhcp snooping` showing **disabled** after workaround.

---

## Suggested recapture order

1. Save `.pkt` with all fixes (Po2 cable, AP port-security, new PCs cabled).
2. `show etherchannel summary` → HA screenshot.
3. Re-export all six `running-config.txt` files.
4. IDF 1-A port-security (Fa0/1 + Fa0/17).
5. Edge Router `show ip dhcp binding` (guest).
6. Guest laptop `ipconfig` + ping isolation/internet.
7. Logical topology labeled PNG (last — shows everything).

---

## New devices to show on diagram (if added)

Per [end-devices.md](../../design/end-devices.md) — up to 3 wired PCs per dept + staff/guest laptops. Label new instances as `PC-DEPT-2`, `Laptop-PT Guest(1)`, etc.

| Department | Example new wired ports |
|---|---|
| Executive | IDF 1-A Fa0/4–8 |
| Finance | IDF 1-A Fa0/12–16 |
| Customer Service | IDF 1-B Fa0/4–14 |
| Operations | IDF 1-B Fa0/18–20 |
| IT | IDF 2-A Fa0/4–8 |
| HR | IDF 2-A Fa0/17–19, IDF 2-B Fa0/13–15 |
| Marketing | IDF 2-B Fa0/4–12 |
| Guest | Wireless only — multiple laptops on AP-GUEST |

---

---

## ⚠️ Unlabeled diagram — LOG zone

The unlabeled topology shows an orange **LOG** box with PCs. **TAMAC has no Logistics/Procurement department** (never VLAN 60). Eight departments only: Executive, Finance, Operations, Customer Service, IT, HR, Marketing, Guest.

**Before submission:** Remove LOG PCs from the `.pkt` or relabel/reassign them to an in-scope department, then recapture unlabeled PNG if needed.

---

*After recapture, update filenames in place or add new numbered files and note in this checklist.*
