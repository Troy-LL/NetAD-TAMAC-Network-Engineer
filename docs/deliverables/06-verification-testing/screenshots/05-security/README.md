# Screenshot index — Phase 5 Security

| Filename | Sub-section | Proves |
|---|---|---|
| `01-core-server-access-acl.png` | ACL | SERVER_ACCESS definition |
| `02-it-ping-dns-allowed.png` | ACL | Staff allowed to DNS |
| `03-guest-ping-dns-denied.png` | ACL | Guest denied DNS |
| `04-edge-guest-restrict-acl.png` | ACL | GUEST_RESTRICT on Gi0/0.90 |
| `05-it-ssh-core-sw.png` | SSH | SSH to core |
| `06-it-ssh-idf2b.png` | SSH | SSH to IDF |
| `07-telnet-blocked.png` | SSH | Telnet disabled |
| `08-ssh-version-vty.png` | SSH | SSH v2 + vty config (optional) |
| `09-guest-internal-pings-blocked.png` | Guest restrictions | Internal pings fail |
| `10-guest-restrict-hit-counts.png` | Guest restrictions | ACL counters |
| `11-idf1a-port-security-status.png` | Port security | Port security enabled |
| `12-port-security-violation.png` | Port security | Err-disabled demo (optional) |
| `13-core-dhcp-snooping.png` | DHCP snooping | Core snooping status |
| `14-idf1a-dhcp-snooping.png` | DHCP snooping | IDF trusted uplinks |
| `15-dhcp-snooping-config.png` | DHCP snooping | Running-config proof |
| `16-password-secret-hash.png` | Password encryption | Type 5 secret hashes |
| `17-username-vty-config.png` | User authentication | Local user + vty |
| `18-ssh-login-authenticated.png` | User authentication | Live SSH login |

See [05-security-testing.md](../05-security-testing.md) for steps.
