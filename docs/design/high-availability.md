# EtherChannel & Rapid STP

[← Source of truth](../../cisco.mdc)

## Mechanisms

- **EtherChannel** (LACP) — dual uplinks per IDF
- **Rapid PVST+** — core is root bridge

## Core switch — EtherChannel

3560-24PS has only **Gi0/1–2**; other uplinks use **Fa0/2–7**. See [ports-and-cabling.md](ports-and-cabling.md).

```
! IDF 1-A — Gi0/1-2
interface range gigabitEthernet0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active

interface port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk

! IDF 1-B — Fa0/2-3 → channel-group 2
! IDF 2-A — Fa0/4-5 → channel-group 3
! IDF 2-B — Fa0/6-7 → channel-group 4
```

## IDF switches (example 1-A)

```
interface range FastEthernet0/23 - 24
 switchport mode trunk
 channel-group 1 mode active

interface port-channel1
 switchport mode trunk
```

| IDF | channel-group |
|---|---|
| 1-A | 1 |
| 1-B | 2 |
| 2-A | 3 |
| 2-B | 4 |

## Rapid STP

```
! Core
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,40,50,70,80,90,100,200 root primary

! All IDFs
spanning-tree mode rapid-pvst
```

Success: `show etherchannel summary` → **PoX(SU)** with **(P)** on both uplink ports.
