# Session 11 — Port security: hacker PC wire-swap demo

**Date:** 2025-06-30  
**Context:** Reiteration for capstone defense — prove wired port security blocks a rogue device when someone steals a desk cable

---

## What this demo proves

A staff PC port is locked to **one MAC address** (sticky port security). If an attacker unplugs the legitimate PC and plugs that same cable into a different machine (“hacker PC”), the switch **shuts the port down** (err-disabled). The link turns **red** — the wire is effectively broken. After the hacker cable is removed and the **original PC** is reconnected, err-disable recovery brings the port back automatically after the timer.

Works on **any department wired PC port** (Executive, Finance, IT, etc.). Use a **wired PC port only** — not an AP uplink.

---

## Topology setup (one-time)

### 1. Legitimate PCs (already built)

Each department has 1–3 wired PCs on IDF access ports — see [end-devices.md](../design/end-devices.md). Example: **PC-EXEC** on IDF 1-A **Fa0/1**.

### 2. Hacker PCs (add in Packet Tracer — one per department)

Add **seven** unconnected PCs (Guest is wireless only — no wired hacker port needed):

| Label | Demo with | IDF | Example port |
|---|---|---|---|
| `HACK-EXEC` | PC-EXEC | 1-A | Fa0/1 |
| `HACK-FIN` | PC-FIN | 1-A | Fa0/9 |
| `HACK-CS` | PC-CUSTSERV | 1-B | Fa0/1 |
| `HACK-OPS` | PC-OPS | 1-B | Fa0/15 |
| `HACK-IT` | PC-IT | 2-A | Fa0/1 |
| `HACK-HR` | PC-HR | 2-A | Fa0/15 |
| `HACK-MARK` | PC-MARK | 2-B | Fa0/1 |

For each:

1. Drag a **PC-PT** into the workspace near that department.
2. Label per table above.
3. **Do not cable it to the switch.** Leave it disconnected — it only receives a stolen cable during the attack step.
4. Set **Desktop → IP Configuration → DHCP** (default is fine).

> The hacker PC is not a second legitimate user. It is a prop for “someone plugged the wrong device into this jack.”

### 3. Switch config — paste on all four IDFs

Your exports already have `port-security` + `sticky` on PC ports, but **`maximum 1`** and **`violation shutdown`** are often missing from `show running-config`. Without them, Packet Tracer may not err-disable on a hacker swap.

**Copy-paste file (all departments):** [port-security-all-departments-paste.txt](../deliverables/02-device-configs/port-security-all-departments-paste.txt)

The paste file also enables automatic recovery:

```
errdisable recovery cause psecure-violation
errdisable recovery interval 30
```

This means the port still goes red when the hacker PC is connected, but after the original PC is reconnected, the switch retries the port automatically after about **30 seconds**.

| IDF | Wired PC ports (max 1, shutdown) | Department(s) |
|---|---|---|
| **1-A** | Fa0/1–16 | Executive Fa0/1–8, Finance Fa0/9–16 |
| **1-B** | Fa0/1–14, Fa0/15–20 | Customer Service, Operations |
| **2-A** | Fa0/1–8, Fa0/15–19 | IT, HR |
| **2-B** | Fa0/1–12, Fa0/13–15, Fa0/17 | Marketing, HR Training |

AP ports on the same IDFs keep **`maximum 10`** (guest **50** on 1-A Fa0/17) and **`violation restrict`** — the paste file sets both in one go.

**After pasting on each switch:**

1. Start **simulation** (play).
2. Confirm every **legitimate PC** link is **green** (sticky MAC learns).
3. Run `show port-security` — wired ports should show **MaxSecureAddr: 1**, **Violation Mode: Shutdown**.

**Prerequisite:** The **legitimate PC must be connected and online first** so the switch learns and sticks its MAC. Check:

```
show port-security interface FastEthernet0/1
```

Expect **SecureStatic Address Aging: 0**, **Maximum MAC Addresses: 1**, **Total MAC Addresses: 1**.

---

## Live demo — attack (wire breaks)

**Example:** PC-EXEC on IDF 1-A Fa0/1, hacker = `HACK-EXEC`.

1. Start **simulation** (play).
2. Confirm **PC-EXEC** link is **green** and has DHCP (`ipconfig` → `192.168.10.x`).
3. Select the **copper cable** from PC-EXEC to Fa0/1.
4. **Delete** the cable (or disconnect from PC-EXEC).
5. Run a new **Copper Straight-Through** from **IDF 1-A Fa0/1** → **HACK-EXEC** FastEthernet0.

**Expected:**

| What you see | Meaning |
|---|---|
| Link turns **red** | Port security violation — port err-disabled |
| HACK-EXEC gets no useful DHCP | Port is down; traffic blocked |
| `show interfaces FastEthernet0/1 status` → **err-disabled** | Violation action = shutdown |

**CLI verify (IDF 1-A):**

```
enable
show port-security interface FastEthernet0/1
show interfaces FastEthernet0/1 status
```

Screenshot: `12-port-security-violation.png` — err-disabled status or red link.

---

## Live demo — recovery (wire fixed)

1. **Delete** the cable from HACK-EXEC.
2. Run a new **Copper Straight-Through** from **Fa0/1** → **PC-EXEC** FastEthernet0 (original PC, original port).
3. Wait for the err-disable recovery timer (about **30 seconds** if using the paste config).
4. Link should turn **green** automatically.
5. On **PC-EXEC**: `ipconfig` / renew DHCP — should get `192.168.10.x` again.
6. `ping 192.168.10.1` — should succeed.

**Expected:** Same sticky MAC as before (`show port-security interface FastEthernet0/1`); violations counter incremented but port is **connected** after recovery.

---

## Repeat on any department

Same steps; only the IDF, port, VLAN, and PC names change.

| Department | Legitimate PC (example) | IDF | Port | Hacker prop |
|---|---|---|---|---|
| Executive | PC-EXEC | 1-A | Fa0/1 | HACK-EXEC |
| Finance | PC-FIN | 1-A | Fa0/9 | HACK-FIN |
| Customer Service | PC-CUSTSERV | 1-B | Fa0/1 | HACK-CS |
| Operations | PC-OPS | 1-B | Fa0/15 | HACK-OPS |
| IT | PC-IT | 2-A | Fa0/1 | HACK-IT |
| HR | PC-HR | 2-A | Fa0/15 | HACK-HR |
| Marketing | PC-MARK | 2-B | Fa0/1 | HACK-MARK |

You can also demo on **PC-EXEC-2** / **Fa0/2**, etc., as long as that port already learned a sticky MAC while the real PC was connected.

---

## Defense one-liner

> “Port security on every wired access port allows only one MAC. If someone unplugs a staff PC and connects a rogue machine to that jack, the switch err-disables the port and the link goes down. Once the rogue device is removed and the authorized PC is reconnected, err-disable recovery brings the link back automatically.”

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Link stays green on hacker PC | Simulation not running; or port security not applied — check `show port-security` |
| No violation after swap | Sticky MAC never learned — connect original PC first, wait, then swap |
| Original PC still red after reconnect | Wait 30 seconds; check `show errdisable recovery`; if PT does not support recovery, use `shutdown` / `no shutdown` as fallback |
| AP port went red | Wrong port — AP ports use `maximum 10–50` + `violation restrict`; demo on **wired PC ports only** |
| New PC on empty port works fine | Expected — first device on an empty port learns sticky; violation only when **wrong** MAC appears |

---

## Related docs

- Config: [security.md](../design/security.md) · [08-security-hardening.md](08-security-hardening.md)
- Verification script: [05-security-testing.md](../deliverables/06-verification-testing/05-security-testing.md) § 5d
- AP contrast (do not confuse): [10-ap-port-security-multi-client.md](10-ap-port-security-multi-client.md)
