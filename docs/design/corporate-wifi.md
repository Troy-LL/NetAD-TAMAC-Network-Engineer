# Corporate Wi‑Fi (Staff)

[← Source of truth](../../cisco.mdc)

Staff laptops use SSID **`TAMAC-Corp`** on department VLANs. Same DHCP and routing as wired PCs in that zone. **No** `GUEST_RESTRICT` ACL.

## AP placement

| AP label | Map location | IDF | Port | VLAN | SSID |
|---|---|---|---|---|---|
| AP-Guest | Lobby (Floor 1) | 1-A | Fa0/17 | 90 | `TAMAC-Guest` |
| AP-EXEC | Executive (Floor 1) | 1-A | Fa0/18 | 10 | `TAMAC-Corp` |
| AP-FIN | Finance (Floor 1) | 1-A | Fa0/19 | 20 | `TAMAC-Corp` |
| AP-CS | Customer Service (Floor 1) | 1-B | Fa0/21 | 40 | `TAMAC-Corp` |
| AP-OPS | Operations (Floor 1) | 1-B | Fa0/22 | 30 | `TAMAC-Corp` |
| AP-IT | IT (Floor 2) | 2-A | Fa0/20 | 50 | `TAMAC-Corp` |
| AP-HR | HR (Floor 2) | 2-A | Fa0/21 | 70 | `TAMAC-Corp` |
| AP-MARK | Marketing (Floor 2) | 2-B | Fa0/16 | 80 | `TAMAC-Corp` |

## AP cabling

- Cable **AP Port 0** → IDF access port (table above)
- Configure wireless on **AP Port 1** (Config tab)
- **Do not** cable Port 1

## Security (Packet Tracer GUI)

| Setting | Guest AP | Corporate APs |
|---|---|---|
| SSID | `TAMAC-Guest` | `TAMAC-Corp` |
| Auth | WPA2-PSK | WPA2-PSK |
| Passphrase | `Guest123!` | `TAMACstaff2024!` |

Use the same passphrase on every corporate AP and on staff laptops.

## Staff laptops (demo)

| Device | Location | Connects to | Expected IP |
|---|---|---|---|
| Laptop-Staff-IT | IT dept | `TAMAC-Corp` via AP-IT | 192.168.50.x |
| Laptop-Staff-CS | Customer Service | `TAMAC-Corp` via AP-CS | 192.168.40.x |
| Laptop-Staff-FIN | Finance | `TAMAC-Corp` via AP-FIN | 192.168.20.x |

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
