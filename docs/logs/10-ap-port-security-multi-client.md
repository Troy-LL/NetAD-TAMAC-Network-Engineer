# Session 10 — AP port security: multiple wireless clients

**Date:** 2025-06-29  
**Context:** Adding guest and staff laptops to APs caused AP uplink ports to go **err-disabled**; new guest laptops showed **DHCP failed**  
**Status:** Resolved

---

## Symptoms

| Symptom | Where |
|---|---|
| AP-GUEST link to IDF 1-A turns **red** (err-disabled) | IDF 1-A Fa0/17 |
| Adding a 2nd+ laptop to IT (or other) AP shuts down the AP | IDF AP access ports |
| Finance laptop cannot ping Finance PC | Same VLAN 20 — check SSID + DHCP first |
| New guest laptops: **DHCP request failed** / `169.254.x.x` | Guest wireless after AP port recovered |

---

## Root cause

**Port security MAC address violation** on AP uplink ports — not an ACL issue.

- Wired PC ports use `maximum 1` MAC (one device per port).
- An AP aggregates **every wireless client MAC** onto its single switch uplink.
- Default `maximum 1` + `violation shutdown` → second client triggers violation → port **err-disabled** → AP appears dead.
- With `maximum 20` on guest AP, **19 DHCP leases + AP MAC** filled the port; device #20+ was blocked → DHCP failed even though Edge Router pool had free addresses.

### Security terms (for defense)

| Term | Applies here? |
|---|---|
| **Port security / MAC violation** | **Yes** — correct term |
| **Err-disabled** | **Yes** — switch shut the AP port |
| **ACL** | **No** — ACL filters IP traffic; it does not shut down switch ports |

---

## Edge Router verification (guest DHCP server OK)

Guest DHCP runs on **Edge Router Gi0/0.90**, not the core.

```
Edge-Router#show ip dhcp pool

Pool GUEST_POOL :
 Total addresses                : 126
 Leased addresses               : 19
 Excluded addresses             : 1
 ...
 192.168.90.1     - 192.168.90.126    19   / 1     / 126

Edge-Router#show ip interface brief
GigabitEthernet0/0.90  192.168.90.1    YES manual up                    up
```

Pool was **not exhausted** (19/126). `Gi0/0.90` was up/up. Failure was L2 on IDF 1-A Fa0/17.

---

## Fix — Guest AP (IDF 1-A Fa0/17)

```text
enable
configure terminal
interface FastEthernet0/17
 switchport mode access
 switchport access vlan 90
 switchport port-security
 switchport port-security maximum 50
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 shutdown
 no shutdown
end
```

**Verified:** Guest laptops received DHCP after this change.

| Setting | Why |
|---|---|
| `maximum 50` | AP MAC + many guest laptops |
| `violation restrict` | Extra MACs dropped; port stays up (no err-disable) |
| `shutdown` / `no shutdown` | Clears existing err-disabled state |

---

## Fix — All corporate AP ports (recommended)

Use `maximum 10` for single-department staff APs; guest uses `50`.

### IDF 1-A

```text
enable
configure terminal
interface FastEthernet0/17
 switchport port-security maximum 50
 switchport port-security violation restrict
 shutdown
 no shutdown
interface FastEthernet0/18
 switchport port-security maximum 10
 switchport port-security violation restrict
 shutdown
 no shutdown
interface FastEthernet0/19
 switchport port-security maximum 10
 switchport port-security violation restrict
 shutdown
 no shutdown
end
```

| Port | AP | VLAN |
|---|---|---|
| Fa0/17 | AP-Guest | 90 |
| Fa0/18 | AP-EXEC | 10 |
| Fa0/19 | AP-FIN | 20 |

### IDF 1-B

```text
enable
configure terminal
interface FastEthernet0/21
 switchport port-security maximum 10
 switchport port-security violation restrict
 shutdown
 no shutdown
interface FastEthernet0/22
 switchport port-security maximum 10
 switchport port-security violation restrict
 shutdown
 no shutdown
end
```

| Port | AP | VLAN |
|---|---|---|
| Fa0/21 | AP-CS | 40 |
| Fa0/22 | AP-OPS | 30 |

### IDF 2-A

```text
enable
configure terminal
interface FastEthernet0/20
 switchport port-security maximum 10
 switchport port-security violation restrict
 shutdown
 no shutdown
interface FastEthernet0/21
 switchport port-security maximum 10
 switchport port-security violation restrict
 shutdown
 no shutdown
end
```

| Port | AP | VLAN |
|---|---|---|
| Fa0/20 | AP-IT | 50 |
| Fa0/21 | AP-HR | 70 |

### IDF 2-B

```text
enable
configure terminal
interface FastEthernet0/16
 switchport port-security maximum 10
 switchport port-security violation restrict
 shutdown
 no shutdown
end
```

| Port | AP | VLAN |
|---|---|---|
| Fa0/16 | AP-MARK | 80 |

---

## Wired PC ports (unchanged)

Keep `maximum 1` + `violation shutdown` on **PC access ports only** (not AP ports).

Example — 2 PCs per department on IDF 1-A:

```text
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security mac-address sticky
 no shutdown
! … Fa0/2 VLAN 10, Fa0/9–10 VLAN 20 …
```

---

## If DHCP still fails after AP fix

1. Confirm laptop on correct SSID (`TAMAC-Guest` → `192.168.90.x`).
2. On CORE-SW + all IDFs: `no ip dhcp snooping` — see [09-dhcp-snooping-pt-workaround.md](09-dhcp-snooping-pt-workaround.md).
3. On Edge Router (guest only): `clear ip dhcp binding *` if PT shows duplicate leases, then `ipconfig /renew` on laptops.

---

## Verify

```text
! On each IDF with an AP
show interfaces status
show port-security interface FastEthernet0/17

! On Edge Router (guest)
show ip dhcp pool
show ip dhcp binding

! On guest laptop
ipconfig
ping 192.168.90.1
ping 10.0.0.1
```

Expected: AP port **connected** (green), not err-disabled; guest gets `192.168.90.x`; internal pings fail, internet OK.

---

## Defense talking point

> Port security on AP uplinks must allow multiple MAC addresses because wireless clients share one switch port. PC ports stay at maximum 1; AP ports use a higher maximum with violation restrict so new clients are controlled without disabling the access point.
