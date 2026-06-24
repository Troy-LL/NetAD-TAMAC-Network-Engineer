# Session 06 — Guest ACL Not Blocking (Troubleshooting)

**Date:** 2025-06-24  
**Issue:** Guest laptop `ping 192.168.10.1` succeeds — should fail.

## Root causes (check in order)

1. **Guest laptop on wrong SSID** (`TAMAC-Corp` instead of `TAMAC-Guest`) → gets dept IP, not 90.x
2. **ACL not applied** on `interface vlan 90` after config reset
3. **ACL line order wrong** (appended lines — `permit any` before `deny`)
4. **Guest AP on wrong VLAN** (not Fa0/17 / VLAN 90)

## Step 1 — Verify guest laptop network

On **Guest laptop** → Command Prompt:

```
ipconfig
```

| Must be | If wrong |
|---|---|
| IP **192.168.90.x** | Wrong SSID or wrong AP VLAN |
| Gateway **192.168.90.1** | Fix AP uplink / DHCP |
| Connected to **TAMAC-Guest** only | Disconnect from TAMAC-Corp in PC Wireless |

**Physical check:** Guest laptop must associate with **AP-Guest** only (dashed wireless line to AP-Guest, not AP-FIN or AP-EXEC).

## Step 2 — Verify Core ACL (CORE-SW)

```
show ip interface vlan 90
show access-lists GUEST_RESTRICT
show running-config
```

Scroll the full `show running-config` and search for `access-group`.

**If ACL lines exist but NO `ip access-group` under any `interface VlanX`**, the ACL was created but never bound. PT CLI often accepts `ip access-group` without saving it — use **Step 3b (GUI)**.

Look for:

```
interface Vlan90
 ip access-group GUEST_RESTRICT in
```

## Step 3 — Recreate ACL and apply (CORE-SW)

**If `show ip interface vlan 90` still shows `Inbound access list is not set` after applying**, use the GUI or the outbound workaround in Step 3b.

### Step 3a — CLI apply (try first)

**Delete and recreate** — do not append:

```
enable
configure terminal
interface vlan 90
 no ip access-group GUEST_RESTRICT in
 no shutdown
no ip access-list extended GUEST_RESTRICT

ip access-list extended GUEST_RESTRICT
 permit udp any any eq bootps
 permit udp any any eq bootpc
 deny   ip 192.168.90.0 0.0.0.127 192.168.10.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.20.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.30.0 0.0.0.127
 deny   ip 192.168.90.0 0.0.0.127 192.168.40.0 0.0.0.127
 deny   ip 192.168.90.0 0.0.0.127 192.168.50.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.70.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.80.0 0.0.0.63
 deny   ip 192.168.90.0 0.0.0.127 192.168.100.0 0.0.0.15
 deny   ip 192.168.90.0 0.0.0.127 192.168.200.0 0.0.0.15
 permit ip 192.168.90.0 0.0.0.127 any

interface vlan 90
 ip access-group GUEST_RESTRICT in
end
```

Verify saved:

```
show running-config interface vlan 90
show ip interface vlan 90
```

Must show `ip access-group GUEST_RESTRICT in` in running-config **and** `Inbound access list is GUEST_RESTRICT`.

### Step 3b — GUI apply (**required if running-config has no access-group lines**)

Your `show running-config` shows `GUEST_RESTRICT` defined but **no** `ip access-group` on any VLAN — CLI did not save. Use the GUI:

1. Click **CORE-SW** → **Config** tab (not CLI).
2. Left panel → **INTERFACE** → **VLAN 90**.
3. **Inbound Access List** dropdown → select **GUEST_RESTRICT**.
4. Optional: **VLAN 100** → **Inbound** → **SERVER_ACCESS**.
5. **File → Save**, then stop simulation → start simulation.
6. CLI check:

```
show running-config
```

Scroll to `interface Vlan90` — must now include:

```
 ip access-group GUEST_RESTRICT in
```

7. `show ip interface vlan 90` should show `Inbound access list is GUEST_RESTRICT` (may still say "not set" in some PT builds — trust running-config + ping test).

### Step 3c — Packet Tracer workaround (outbound on internal SVIs)

If inbound on Vlan90 never sticks, block guest traffic leaving toward internal VLANs:

```
enable
configure terminal
ip access-list extended GUEST_BLOCK_OUT
 deny ip 192.168.90.0 0.0.0.127 any
 permit ip any any

interface vlan 10
 ip access-group GUEST_BLOCK_OUT out
interface vlan 20
 ip access-group GUEST_BLOCK_OUT out
interface vlan 30
 ip access-group GUEST_BLOCK_OUT out
interface vlan 40
 ip access-group GUEST_BLOCK_OUT out
interface vlan 50
 ip access-group GUEST_BLOCK_OUT out
interface vlan 70
 ip access-group GUEST_BLOCK_OUT out
interface vlan 80
 ip access-group GUEST_BLOCK_OUT out
interface vlan 100
 ip access-group GUEST_BLOCK_OUT out
end
```

Do **not** apply on **Vlan 200** — guest needs that path to reach the router/internet.

## Step 4 — Verify guest AP port (IDF 1-A)

```
show vlan brief
show interfaces FastEthernet0/17 switchport
```

Fa0/17 must be **access VLAN 90**.

If not:

```
configure terminal
interface FastEthernet0/17
 switchport mode access
 switchport access vlan 90
 no shutdown
end
```

## Step 5 — Re-test

Stop simulation → start simulation. On **Guest laptop**:

```
ipconfig
ping 192.168.90.1
ping 192.168.10.1
ping 10.0.0.1
```

| Test | Expected |
|---|---|
| ping 192.168.90.1 | OK (gateway) |
| ping 192.168.10.1 | **Fail** |
| ping 10.0.0.1 | OK (internet) |

## Step 6 — Confirm staff still works

On **STAFF-FIN**:

```
ping 192.168.10.1
```

Must still **succeed** (staff is not on VLAN 90).
