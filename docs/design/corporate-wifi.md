# Corporate Wi‑Fi (Staff)

[← Source of truth](../../cisco.mdc)

Staff laptops connect to **department-specific SSIDs** (`TAMAC-Corp-<dept>`) on their department VLAN. Each SSID maps to one VLAN so devices auto-join the right network. Same DHCP and routing as wired PCs in that zone. **No** `GUEST_RESTRICT` ACL.

## AP placement

| AP label | Map location | IDF | Port | VLAN | SSID |
|---|---|---|---|---|---|
| AP-Guest | Lobby (Floor 1) | 1-A | Fa0/17 | 90 | `TAMAC-Guest` |
| AP-EXEC | Executive (Floor 1) | 1-A | Fa0/18 | 10 | `TAMAC-Corp-exec` |
| AP-FIN | Finance (Floor 1) | 1-A | Fa0/19 | 20 | `TAMAC-Corp-fin` |
| AP-CS | Customer Service (Floor 1) | 1-B | Fa0/21 | 40 | `TAMAC-Corp-cs` |
| AP-OPS | Operations (Floor 1) | 1-B | Fa0/22 | 30 | `TAMAC-Corp-ops` |
| AP-IT | IT (Floor 2) | 2-A | Fa0/20 | 50 | `TAMAC-Corp-it` |
| AP-HR | HR (Floor 2) | 2-A | Fa0/21 | 70 | `TAMAC-Corp-hr` |
| AP-MARK | Marketing (Floor 2) | 2-B | Fa0/16 | 80 | `TAMAC-Corp-mark` |

## AP cabling

- Cable **AP Port 0** → IDF access port (table above)
- Configure wireless on **AP Port 1** (Config tab)
- **Do not** cable Port 1

### AP switch port security (required for multiple laptops)

Wireless clients share the AP uplink. On each AP port, raise port-security maximum — see [security.md](security.md) and [log 10](../logs/10-ap-port-security-multi-client.md).

Guest AP (IDF 1-A Fa0/17) example:

```text
interface FastEthernet0/17
 switchport port-security maximum 50
 switchport port-security violation restrict
```

Staff APs: `maximum 10`, same `violation restrict`.

## Security (Packet Tracer GUI)

| Setting | Guest AP | Corporate APs |
|---|---|---|
| SSID | `TAMAC-Guest` | `TAMAC-Corp-<dept>` (per department) |
| Auth | WPA2-PSK | WPA2-PSK |
| Passphrase | `Guest123!` | `TAMACstaff2024!` |

Each corporate AP broadcasts its own department SSID, but all share the same passphrase and staff laptops use the same passphrase.

## Staff laptops (demo)

| Device | Location | Connects to | Expected IP |
|---|---|---|---|
| Laptop-Staff-IT | IT dept | `TAMAC-Corp-it` via AP-IT | 192.168.50.x |
| Laptop-Staff-CS | Customer Service | `TAMAC-Corp-cs` via AP-CS | 192.168.40.x |
| Laptop-Staff-FIN | Finance | `TAMAC-Corp-fin` via AP-FIN | 192.168.20.x |

Laptop setup: Physical → power off → install **WPC300N** → power on → Desktop → PC Wireless.

## Traffic flow

```
Staff laptop → AP (dept VLAN) → IDF → Core → internal + internet OK
Guest laptop → AP (VLAN 90) → Core → GUEST_RESTRICT → internet only
```

## Minimum for capstone demo

- **Guest:** AP-Guest + guest laptop (already done)
- **Staff:** AP-IT + one staff laptop in IT

Optional: AP-CS + staff laptop in Customer Service (highest-density department).
