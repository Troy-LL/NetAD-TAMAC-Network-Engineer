# Packet Tracer Guide & Checklists

[← Source of truth](../../cisco.mdc)

## Setup order

1. Place devices → cable core path first (ISP, router, core, DNS)
2. Configure core (VLANs, STP root, `ip routing`, SVIs) before IDF uplinks
3. EtherChannel on core + each IDF
4. Access ports on IDFs
5. Router WAN + NAT
6. PCs → DHCP test
7. Corporate APs (`TAMAC-Corp`) + staff laptops — see [end-devices.md](end-devices.md)

## CLI tips

| Issue | Fix |
|---|---|
| `vlan 10,20,30` | One `vlan X` per line |
| Config from `>` prompt | `en` then `config t` |
| ACL edit breaks order | `no ip access-list extended NAME` then recreate |
| No serial port on 2911 | Add HWIC-2T, power cycle |

## NAT (edge router)

```
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 interface Serial0/0/0 overload

interface Serial0/0/0
 ip nat outside
interface GigabitEthernet0/0
 ip nat inside
```

## Defense testing checklist

- [ ] DHCP correct per VLAN (IP, gateway, DNS)
- [ ] Same-VLAN ping
- [ ] Inter-VLAN ping
- [ ] Ping DNS 192.168.100.2
- [ ] Ping internet from internal PC
- [ ] Guest → internet OK; guest → internal blocked
- [ ] Staff laptop on `TAMAC-Corp` → DHCP + ping DNS + inter-VLAN OK
- [ ] Staff laptop ≠ guest subnet (50.x vs 90.x)
- [ ] Extra PCs per dept get DHCP on correct VLAN
- [ ] SSH to core and IDF from IT PC
- [ ] Telnet rejected
- [ ] EtherChannel survives one unplugged uplink
- [ ] Port security violation on rogue MAC
- [ ] DHCP snooping blocks rogue server

## Documentation

- Screenshot every test
- Export `show running-config` from each device
- Session logs → [docs/logs/](../logs/)
