# SNMP Configuration — Cisco Switch
> A guide to configuring SNMP on Cisco switches for network monitoring and management.

**Category:** Networking  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **SNMP** | Simple Network Management Protocol | A protocol for monitoring and managing network devices |
| **NMS** | Network Management System | Software that polls SNMP agents e.g. Zabbix, NetBox, PRTG |
| **Agent** | SNMP Agent | Software running on the network device that responds to SNMP queries |
| **MIB** | Management Information Base | A database of objects that can be monitored via SNMP |
| **OID** | Object Identifier | A unique numeric ID for each item in the MIB e.g. interface status |
| **Community String** | Community String | A password-like string used for SNMP v1 and v2c authentication |
| **Trap** | SNMP Trap | An unsolicited alert sent FROM the device TO the NMS when something happens |
| **Inform** | SNMP Inform | Like a trap but requires acknowledgement from the NMS |
| **Poll** | SNMP Poll | The NMS actively querying the device for data |
| **GET** | SNMP GET | A request to retrieve a specific OID value |
| **GETBULK** | SNMP GETBULK | A request to retrieve multiple OID values efficiently |
| **SET** | SNMP SET | A request to change a value on the device (write access) |
| **v1** | SNMP Version 1 | Original version — no encryption, no authentication beyond community string |
| **v2c** | SNMP Version 2c | Adds GETBULK and 64-bit counters — still uses community strings |
| **v3** | SNMP Version 3 | Adds authentication and encryption — use in production |
| **CDP** | Cisco Discovery Protocol | Cisco's Layer 2 protocol for discovering neighboring Cisco devices |

---

## Overview
SNMP allows network monitoring tools like Zabbix, Argus Ops, PRTG, or SolarWinds to:
- Monitor interface status (up/down)
- Track bandwidth utilization
- Read CPU and memory usage
- Discover connected devices via CDP/LLDP
- Receive alerts when something goes wrong (traps)

**SNMP Version Comparison:**

| Feature | v1 | v2c | v3 |
|---------|-----|-----|-----|
| Authentication | Community string | Community string | Username + password |
| Encryption | None | None | AES/DES |
| GETBULK | No | Yes | Yes |
| 64-bit counters | No | Yes | Yes |
| Recommended | No | Lab only | Yes — production |

---

## Prerequisites
- Cisco Catalyst switch with IOS
- Console or SSH access
- SNMP monitoring tool configured (Zabbix, Argus Ops, etc.)
- Management IP configured on the switch

---

## 1. Configuring SNMP v2c (Read-Only)

SNMPv2c is commonly used in lab environments and internal networks where encryption is less critical.

```ios
SW-CORE-01# configure terminal

! Create a read-only community string
SW-CORE-01(config)# snmp-server community PUBLIC_STRING ro

! Restrict SNMP to a specific management host (recommended)
SW-CORE-01(config)# access-list 10 permit 192.168.99.10

! Apply ACL to community string
SW-CORE-01(config)# snmp-server community PUBLIC_STRING ro 10

! Set system information
SW-CORE-01(config)# snmp-server location "Server Room - Rack A"
SW-CORE-01(config)# snmp-server contact "Brady Genik - brady@company.com"

! Save
SW-CORE-01# wr
```

> ⚠️ **Security Note:** Never use `public` or `private` as community strings in production — they are the default and well-known to attackers.

---

## 2. Configuring SNMP v3 (Recommended for Production)

SNMPv3 adds authentication and encryption — use this in any environment handling sensitive data.

```ios
! Create an SNMP group with authentication and privacy (encryption)
SW-CORE-01(config)# snmp-server group MONITOR_GROUP v3 priv

! Create an SNMP user in that group
! auth sha — SHA authentication (more secure than MD5)
! priv aes 128 — AES-128 encryption
SW-CORE-01(config)# snmp-server user MONITOR_USER MONITOR_GROUP v3 auth sha AuthPass123 priv aes 128 PrivPass123

! Set system info
SW-CORE-01(config)# snmp-server location "Server Room - Rack A"
SW-CORE-01(config)# snmp-server contact "Brady Genik - brady@company.com"

! Save
SW-CORE-01# wr

! Verify user was created
SW-CORE-01# show snmp user
```

---

## 3. Configuring SNMP Traps

Traps send alerts to your NMS when events occur — interface goes down, high CPU, etc.

```ios
! Configure trap destination (NMS IP address)
SW-CORE-01(config)# snmp-server host 192.168.99.10 version 2c PUBLIC_STRING

! Or for SNMPv3
SW-CORE-01(config)# snmp-server host 192.168.99.10 version 3 priv MONITOR_USER

! Enable specific trap types
SW-CORE-01(config)# snmp-server enable traps snmp linkdown linkup
SW-CORE-01(config)# snmp-server enable traps config
SW-CORE-01(config)# snmp-server enable traps envmon temperature
SW-CORE-01(config)# snmp-server enable traps port-security

! Enable all traps (noisy — use specific traps instead in production)
SW-CORE-01(config)# snmp-server enable traps

! Save
SW-CORE-01# wr
```

---

## 4. Enabling CDP for Neighbor Discovery

CDP (Cisco Discovery Protocol) allows monitoring tools to discover what's connected to each port.

```ios
! Enable CDP globally (usually enabled by default)
SW-CORE-01(config)# cdp run

! Enable CDP on a specific interface
SW-CORE-01(config)# interface GigabitEthernet 0/1
SW-CORE-01(config-if)# cdp enable
SW-CORE-01(config-if)# exit

! Disable CDP on untrusted edge ports (security best practice)
SW-CORE-01(config)# interface range FastEthernet 0/1-24
SW-CORE-01(config-if-range)# no cdp enable
SW-CORE-01(config-if-range)# exit

! Verify CDP neighbors
SW-CORE-01# show cdp neighbors
SW-CORE-01# show cdp neighbors detail
```

---

## 5. Useful SNMP OIDs for Monitoring

These are common OIDs used by monitoring tools to poll Cisco switches:

| OID | Description |
|-----|-------------|
| `1.3.6.1.2.1.1.1.0` | System description |
| `1.3.6.1.2.1.1.3.0` | System uptime |
| `1.3.6.1.2.1.1.5.0` | System hostname |
| `1.3.6.1.2.1.2.2.1.8` | Interface operational status (1=up, 2=down) |
| `1.3.6.1.2.1.2.2.1.10` | Interface inbound octets (bytes received) |
| `1.3.6.1.2.1.2.2.1.16` | Interface outbound octets (bytes sent) |
| `1.3.6.1.2.1.2.2.1.14` | Interface inbound errors |
| `1.3.6.1.4.1.9.2.1.58.0` | Cisco CPU utilization (5-min average) |
| `1.3.6.1.4.1.9.9.48.1.1.1.5` | Cisco memory pool used |
| `1.3.6.1.4.1.9.9.48.1.1.1.6` | Cisco memory pool free |

---

## 6. Testing SNMP from Linux/Windows

### Linux
```bash
# Install SNMP tools
sudo apt install snmp snmp-mibs-downloader

# Test SNMPv2c — get system description
snmpget -v2c -c PUBLIC_STRING 192.168.1.10 1.3.6.1.2.1.1.1.0

# Walk all OIDs (SNMP walk)
snmpwalk -v2c -c PUBLIC_STRING 192.168.1.10

# Test SNMPv3
snmpget -v3 -u MONITOR_USER -l authPriv \
  -a SHA -A AuthPass123 \
  -x AES -X PrivPass123 \
  192.168.1.10 1.3.6.1.2.1.1.1.0
```

### Windows PowerShell
```powershell
# Test SNMP connectivity (requires SNMP tools installed)
# Or use a tool like Net-SNMP for Windows
```

---

## 7. Verifying SNMP Configuration

```ios
! View SNMP configuration summary
SW-CORE-01# show snmp

! View SNMP community strings
SW-CORE-01# show snmp community

! View SNMP users (v3)
SW-CORE-01# show snmp user

! View SNMP groups (v3)
SW-CORE-01# show snmp group

! View trap destinations
SW-CORE-01# show snmp host

! View SNMP statistics
SW-CORE-01# show snmp statistics

! View CDP neighbors
SW-CORE-01# show cdp neighbors detail
```

---

## 8. Removing SNMP Configuration

```ios
! Remove a community string
SW-CORE-01(config)# no snmp-server community PUBLIC_STRING

! Remove an SNMPv3 user
SW-CORE-01(config)# no snmp-server user MONITOR_USER MONITOR_GROUP v3

! Remove a trap destination
SW-CORE-01(config)# no snmp-server host 192.168.99.10

! Disable SNMP entirely
SW-CORE-01(config)# no snmp-server
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| NMS can't reach switch via SNMP | Firewall blocking UDP 161 or wrong community | Check firewall, verify community string matches on both sides |
| SNMP walk returns nothing | ACL blocking NMS IP | Check ACL applied to community string |
| SNMPv3 authentication fails | Wrong auth/priv passwords or protocol mismatch | Verify user config with `show snmp user`, recreate if needed |
| Traps not being received | Wrong trap host IP or traps not enabled | Check `show snmp host` and `snmp-server enable traps` |
| Interface OIDs not returning data | Interface index changed after reboot | Re-discover the device in your NMS |
| CDP neighbors not showing | CDP disabled globally or on interface | Run `cdp run` globally and `cdp enable` on interface |

---

## Quick Reference

```ios
! Add SNMPv2c read-only community
snmp-server community YOUR_STRING ro

! Add SNMPv3 group and user
snmp-server group MONITOR_GROUP v3 priv
snmp-server user MONITOR_USER MONITOR_GROUP v3 auth sha AuthPass priv aes 128 PrivPass

! Set location and contact
snmp-server location "Location"
snmp-server contact "Name - email"

! Add trap destination
snmp-server host 192.168.99.10 version 2c YOUR_STRING

! Enable link traps
snmp-server enable traps snmp linkdown linkup

! Show SNMP config
show snmp
show snmp user
show snmp host

! Show CDP neighbors
show cdp neighbors detail
```

---

## Notes
- SNMP uses UDP port **161** for queries and port **162** for traps
- Always use SNMPv3 with authentication and privacy in production environments
- Use a strong community string — never use `public` or `private`
- Restrict SNMP access to your NMS IP using an ACL
- CDP should be disabled on ports facing untrusted networks — it reveals device type and IOS version
- Argus Ops uses SNMP GETBULK to efficiently collect port status, MAC tables, and CDP neighbor data

---

## Related Documents
- [Cisco Switch Configuration](cisco-switch-configuration.md)
- [VLAN Configuration](vlan-configuration-cisco-switch.md)
- [Cisco Switch Troubleshooting](../Hardware/cisco-switch-troubleshooting.md)
