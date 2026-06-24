# Session 09 — DHCP snooping Packet Tracer workaround

**Date:** 2025-06-25  
**Context:** Documentation session — DHCP failed on staff PCs during screenshot capture  
**Status:** Resolved for lab use

---

## Symptom

Executive PC DHCP failed (`DHCP request failed`) even though:

- IDF-1A ports were up (no port-security violation)
- CORE-SW `Vlan10` was up/up
- `EXEC_POOL` existed with available addresses
- Static IP `192.168.10.50` could ping gateway `192.168.10.1`

## Root cause

DHCP snooping on CORE-SW dropped DISCOVER packets with Option 82 from untrusted port-channels:

```text
%DHCP_SNOOPING-5-DHCP_SNOOPING_NONZERO_GIADDR: DHCP_SNOOPING drop message ...
on untrusted port, message type: DHCP DISCOVER
```

Packet Tracer will not apply `ip dhcp snooping trust` to `Port-channel1–4`. Trusting physical member ports alone was not enough.

## Fix applied (documentation / lab runtime)

On **CORE-SW** and all **IDFs**:

```text
configure terminal
no ip dhcp snooping
end
```

DHCP then worked on staff PCs and guest laptop.

## Notes for defense

- Running configs in `docs/deliverables/02-device-configs/` still show DHCP snooping **configured** (design intent).
- For live DHCP in Packet Tracer, snooping may need to stay disabled unless using `ip dhcp snooping information option allow-untrusted` on the core.
- Port security, SSH, ACLs, and EtherChannel are unaffected.

## Verified after fix

| Test | Result |
|---|---|
| EXEC PC DHCP | `192.168.10.32`, GW `.10.1`, DNS `.100.2` |
| Guest laptop DHCP | `192.168.90.6`, GW `.90.1`, DNS `8.8.8.8` |
| EXEC ping DNS | OK |
| EXEC ping Finance GW | OK |
| EXEC ping internet | OK |
| Guest isolation | Internal blocked, internet OK |
