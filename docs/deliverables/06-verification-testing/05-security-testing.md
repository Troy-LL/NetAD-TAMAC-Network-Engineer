# Phase 5 — Security Testing

**Prerequisites:** [Phase 4](04-dhcp-verification.md) complete.  
**Screenshot folder:** `screenshots/05-security/`

Complete **all seven** sub-sections (5a–5g) before [Phase 6](06-guest-network-testing.md). Guest *behavior* is tested end-to-end in phase 6; here you prove the security *controls exist and work*.

---

## 5a — Access Control Lists

### SERVER_ACCESS (DNS protection on Core)

**Where:** **CORE-SW** → **CLI** tab

```
enable
show access-lists SERVER_ACCESS
show running-config | section access-list
```

**Expected:** Permits dept subnets → `192.168.100.0/28`; deny any → server subnet; permit any any at end.

**Functional test — staff allowed:**

**Where:** **PC-IT** → **Desktop** → **Command Prompt**

```
ping 192.168.100.2
```

**Expected:** Success.

**Functional test — guest denied (preview for phase 6):**

**Where:** **Guest laptop** → **Command Prompt**

```
ping 192.168.100.2
```

**Expected:** Request timeout / unreachable from `192.168.90.1`.

**Screenshots:**

| File | Content |
|---|---|
| `01-core-server-access-acl.png` | `show access-lists SERVER_ACCESS` on CORE-SW |
| `02-it-ping-dns-allowed.png` | PC-IT ping DNS success |
| `03-guest-ping-dns-denied.png` | Guest ping DNS failure |

> **PT quirk:** `show ip interface vlan 100` may show inbound ACL **not set** on the SVI. ACL is still in running-config; guest block is also enforced by **GUEST_RESTRICT** on the router.

---

### GUEST_RESTRICT (guest isolation on Edge Router)

**Where:** **Edge Router** → **CLI** tab

```
show access-lists GUEST_RESTRICT
show ip interface GigabitEthernet0/0.90
```

**Expected:** ACL applied **inbound** on `Gi0/0.90`; deny lines for all internal subnets; permit bootp + internet.

**Screenshot:** `04-edge-guest-restrict-acl.png`

---

## 5b — Secure Remote Access (SSH)

**Where:** **PC-IT** → **Desktop** → **Command Prompt**

SSH to core (password `TamacAdmin2024!` when prompted):

```
ssh -l admin 192.168.200.1
```

After login, type `exit`. Then SSH to an IDF:

```
ssh -l admin 192.168.200.5
```

Use password `Tamac2024` for IDF 2-B.

**Prove telnet is blocked:**

```
telnet 192.168.200.1
```

**Expected:** Connection fails or is refused.

**Screenshots:**

| File | Content |
|---|---|
| `05-it-ssh-core-sw.png` | Successful SSH session to CORE-SW |
| `06-it-ssh-idf2b.png` | Successful SSH to IDF 2-B |
| `07-telnet-blocked.png` | Telnet failure message |

**Config proof (optional):**

**Where:** **CORE-SW** → **CLI**

```
show ip ssh
show running-config | section vty
```

**Screenshot:** `08-ssh-version-vty.png`

---

## 5c — Guest Network Restrictions (ACL proof)

This sub-test documents the **control**; full user journey is in phase 6.

**Where:** **Guest laptop** → **Command Prompt**

```
ping 192.168.10.1
ping 192.168.50.1
ping 192.168.200.1
```

**Expected:** All internal pings fail (unreachable from `192.168.90.1`).

**Where:** **Edge Router** → **CLI**

```
show access-lists GUEST_RESTRICT
```

**Screenshot:** `09-guest-internal-pings-blocked.png`  
**Screenshot:** `10-guest-restrict-hit-counts.png` — ACL with match counters if visible after pings.

---

## 5d — Port Security

### Show configuration

**Where:** **IDF 1-A** → **CLI** tab

```
enable
show port-security
show port-security interface FastEthernet0/1
```

**Expected on PC port Fa0/1:** Port Security enabled, max 1, violation shutdown, sticky MAC.

**Screenshot:** `11-idf1a-port-security-status.png`

### Violation demo (optional but strong proof)

1. Note which PC is on **IDF 1-A Fa0/1** (PC-EXEC).
2. Disconnect **PC-EXEC**, connect a **different PC** to Fa0/1.
3. **Where:** **IDF 1-A** → **CLI**:

```
show interfaces FastEthernet0/1 status
```

**Expected:** Port **err-disabled**.

4. Restore: reconnect PC-EXEC, then on **IDF 1-A** CLI:

```
configure terminal
interface FastEthernet0/1
 shutdown
 no shutdown
end
```

**Screenshot:** `12-port-security-violation.png` — err-disabled port or status output.

> AP ports use higher `maximum` (guest 50, staff 10) — see [log 10](../../logs/10-ap-port-security-multi-client.md). Demo violation on a **wired PC port**, not an AP port.

---

## 5e — DHCP Snooping

**Where:** **CORE-SW** → **CLI** tab

```
show ip dhcp snooping
show ip dhcp snooping binding
```

**Expected (design intent in running-config):** Snooping enabled; trusted uplinks on Fa0/1, Fa0/2–8, Gi0/1–2.

**Where:** **IDF 1-A** → **CLI**

```
show ip dhcp snooping
```

**Expected:** Trusted Fa0/23–24.

**Screenshots:**

| File | Content |
|---|---|
| `13-core-dhcp-snooping.png` | Core snooping summary |
| `14-idf1a-dhcp-snooping.png` | IDF trusted uplinks |

**Defense note:** For live DHCP in Packet Tracer, snooping may be **disabled at runtime** — [log 09](../../logs/09-dhcp-snooping-pt-workaround.md). Screenshots of **running-config** still prove design; mention PT workaround in presentation.

**Config proof:**

**Where:** **CORE-SW** → **CLI**

```
show running-config | include dhcp snooping
```

**Screenshot:** `15-dhcp-snooping-config.png`

---

## 5f — Password Encryption

Passwords use **`secret`** (MD5 type 5 hash) for enable and local users — not plain text.

**Where:** **CORE-SW** → **CLI** tab

```
show running-config | include secret
show running-config | include password-encryption
```

**Expected:** Lines like `enable secret 5 $1$...` and `username admin secret 5 $1$...` — **not** plain `TamacAdmin2024!` in running-config.

**Screenshot:** `16-password-secret-hash.png`

**Enable encryption for line passwords (if configured):**

```
service password-encryption
```

Show before/after if your rubric requires `service password-encryption` explicitly.

---

## 5g — User Authentication

Proves local user database + SSH login (not telnet / not no-login).

**Where:** **CORE-SW** → **CLI**

```
show running-config | section username
show running-config | section line vty
```

**Expected:**

- `username admin privilege 15 secret ...`
- `login local` under `line vty 0 4`
- `transport input ssh` (telnet disabled)

**Live login test:**

**Where:** **PC-IT** → **Command Prompt**

```
ssh -l admin 192.168.200.1
```

Enter password when prompted — must reach `#` prompt after `enable`.

**Screenshots:**

| File | Content |
|---|---|
| `17-username-vty-config.png` | Username + vty lines from running-config |
| `18-ssh-login-authenticated.png` | Successful authenticated SSH session |

---

## Phase 5 gate

| Sub-section | Minimum screenshots |
|---|---|
| 5a ACL | 4 files (01–04) |
| 5b SSH | 3 files (05–07) |
| 5c Guest restrictions | 2 files (09–10) |
| 5d Port security | 1–2 files (11–12) |
| 5e DHCP snooping | 2–3 files (13–15) |
| 5f Password encryption | 1 file (16) |
| 5g User authentication | 2 files (17–18) |

All functional tests must match expected pass/fail before phase 6.

**Next:** [Phase 6 — Guest Network Testing](06-guest-network-testing.md)
