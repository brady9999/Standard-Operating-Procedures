# VLAN Configuration — Cisco Switch
> A guide to creating, managing, and troubleshooting VLANs on Cisco Catalyst switches.

**Category:** Networking  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **VLAN** | Virtual Local Area Network | A logical segmentation of a network — devices on different VLANs can't communicate without a router |
| **Access Port** | Access Port | A port assigned to exactly one VLAN — used for end devices |
| **Trunk Port** | Trunk Port | A port that carries traffic for multiple VLANs simultaneously |
| **Native VLAN** | Native VLAN | The VLAN that carries untagged traffic on a trunk port (default VLAN 1) |
| **Allowed VLANs** | Allowed VLANs | The list of VLANs permitted to traverse a trunk port |
| **802.1Q** | IEEE 802.1Q | The standard protocol for VLAN tagging on trunk ports |
| **VLAN Tag** | VLAN Tag | A 4-byte header added to Ethernet frames identifying which VLAN they belong to |
| **Inter-VLAN Routing** | Inter-VLAN Routing | Routing traffic between VLANs using a router or Layer 3 switch |
| **SVI** | Switched Virtual Interface | A virtual interface on a Layer 3 switch used for inter-VLAN routing |
| **DTP** | Dynamic Trunking Protocol | A Cisco protocol that automatically negotiates trunk links — should be disabled in production |
| **VTP** | VLAN Trunking Protocol | A Cisco protocol that synchronizes VLAN databases across switches — use with caution |
| **PVID** | Port VLAN ID | The VLAN ID assigned to an access port |

---

## Overview
VLANs segment a physical network into multiple logical networks. Even though all devices share the same physical switch, devices on different VLANs cannot communicate with each other without a router — providing security and traffic isolation.

**Common VLAN use cases:**
- Separating departments (IT, HR, Finance)
- Isolating guest WiFi from corporate network
- Separating voice (VoIP) from data traffic
- Management VLAN for network devices
- IoT device isolation

**Example VLAN design:**
```
VLAN 10  — IT Department        192.168.10.0/24
VLAN 20  — HR Department        192.168.20.0/24
VLAN 30  — Finance              192.168.30.0/24
VLAN 40  — Guest WiFi           192.168.40.0/24
VLAN 99  — Management           192.168.99.0/24
VLAN 1   — Default (avoid using)
```

---

## Prerequisites
- Cisco Catalyst switch with IOS
- Console or SSH access
- Basic familiarity with Cisco CLI (see Cisco Switch Configuration doc)

---

## 1. Creating VLANs

```ios
SW-CORE-01# configure terminal

! Create a single VLAN
SW-CORE-01(config)# vlan 10
SW-CORE-01(config-vlan)# name IT-Department
SW-CORE-01(config-vlan)# exit

! Create multiple VLANs at once
SW-CORE-01(config)# vlan 20
SW-CORE-01(config-vlan)# name HR-Department
SW-CORE-01(config-vlan)# exit

SW-CORE-01(config)# vlan 30
SW-CORE-01(config-vlan)# name Finance
SW-CORE-01(config-vlan)# exit

SW-CORE-01(config)# vlan 99
SW-CORE-01(config-vlan)# name Management
SW-CORE-01(config-vlan)# exit

! Verify VLANs were created
SW-CORE-01# show vlan brief
```

---

## 2. Configuring Access Ports

Access ports connect end devices (PCs, printers, phones) and carry traffic for exactly one VLAN.

```ios
! Assign a single port to VLAN 10
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# description IT-PC-01
SW-CORE-01(config-if)# switchport mode access
SW-CORE-01(config-if)# switchport access vlan 10
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

! Assign a range of ports to a VLAN
SW-CORE-01(config)# interface range FastEthernet 0/1-8
SW-CORE-01(config-if-range)# switchport mode access
SW-CORE-01(config-if-range)# switchport access vlan 10
SW-CORE-01(config-if-range)# no shutdown
SW-CORE-01(config-if-range)# exit

! Assign different port ranges to different VLANs
SW-CORE-01(config)# interface range FastEthernet 0/9-16
SW-CORE-01(config-if-range)# switchport mode access
SW-CORE-01(config-if-range)# switchport access vlan 20
SW-CORE-01(config-if-range)# exit

SW-CORE-01(config)# interface range FastEthernet 0/17-24
SW-CORE-01(config-if-range)# switchport mode access
SW-CORE-01(config-if-range)# switchport access vlan 30
SW-CORE-01(config-if-range)# exit

! Save
SW-CORE-01# wr
```

---

## 3. Configuring Trunk Ports

Trunk ports carry traffic for multiple VLANs and are used to connect:
- Switch to switch
- Switch to router
- Switch to wireless access point

```ios
! Configure a trunk port
SW-CORE-01(config)# interface GigabitEthernet 0/1
SW-CORE-01(config-if)# description Trunk-to-Distribution-SW
SW-CORE-01(config-if)# switchport mode trunk

! On older IOS you may need to set encapsulation first
SW-CORE-01(config-if)# switchport trunk encapsulation dot1q

! Allow only specific VLANs on the trunk (best practice — don't allow all)
SW-CORE-01(config-if)# switchport trunk allowed vlan 10,20,30,99

! Set the native VLAN (untagged traffic) — should NOT be VLAN 1 for security
SW-CORE-01(config-if)# switchport trunk native vlan 99

SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

! Verify trunk configuration
SW-CORE-01# show interfaces trunk
SW-CORE-01# show interfaces GigabitEthernet 0/1 trunk
```

---

## 4. Modifying Trunk Allowed VLANs

```ios
! Add a VLAN to the allowed list
SW-CORE-01(config)# interface GigabitEthernet 0/1
SW-CORE-01(config-if)# switchport trunk allowed vlan add 40

! Remove a VLAN from the allowed list
SW-CORE-01(config-if)# switchport trunk allowed vlan remove 40

! Allow all VLANs (not recommended for production)
SW-CORE-01(config-if)# switchport trunk allowed vlan all

! Reset to default (all VLANs allowed)
SW-CORE-01(config-if)# no switchport trunk allowed vlan
```

---

## 5. Configuring the Management VLAN

The management VLAN provides remote access to the switch. Change it from the default VLAN 1 for security.

```ios
! Create management VLAN
SW-CORE-01(config)# vlan 99
SW-CORE-01(config-vlan)# name Management
SW-CORE-01(config-vlan)# exit

! Assign IP to management VLAN interface
SW-CORE-01(config)# interface vlan 99
SW-CORE-01(config-if)# ip address 192.168.99.10 255.255.255.0
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

! Set default gateway
SW-CORE-01(config)# ip default-gateway 192.168.99.1

! Remove IP from VLAN 1 (default)
SW-CORE-01(config)# interface vlan 1
SW-CORE-01(config-if)# no ip address
SW-CORE-01(config-if)# shutdown
SW-CORE-01(config-if)# exit
```

---

## 6. Disabling DTP (Dynamic Trunking Protocol)

DTP automatically negotiates trunk links — disable it on all ports for security.

```ios
! Disable DTP on access ports
SW-CORE-01(config)# interface range FastEthernet 0/1-24
SW-CORE-01(config-if-range)# switchport nonegotiate
SW-CORE-01(config-if-range)# exit

! Disable DTP on trunk ports
SW-CORE-01(config)# interface GigabitEthernet 0/1
SW-CORE-01(config-if)# switchport nonegotiate
SW-CORE-01(config-if)# exit
```

---

## 7. Inter-VLAN Routing (Layer 3 Switch)

On a Layer 3 switch you can route between VLANs using SVIs (Switched Virtual Interfaces) without a separate router.

```ios
! Enable IP routing
SW-CORE-01(config)# ip routing

! Create SVI for each VLAN
SW-CORE-01(config)# interface vlan 10
SW-CORE-01(config-if)# ip address 192.168.10.1 255.255.255.0
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

SW-CORE-01(config)# interface vlan 20
SW-CORE-01(config-if)# ip address 192.168.20.1 255.255.255.0
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

SW-CORE-01(config)# interface vlan 30
SW-CORE-01(config-if)# ip address 192.168.30.1 255.255.255.0
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

! Verify routing table
SW-CORE-01# show ip route
```

---

## 8. Router-on-a-Stick (Inter-VLAN via Router)

If using a regular Layer 2 switch with a router, configure subinterfaces on the router for each VLAN.

```ios
! On the router — configure subinterfaces
Router(config)# interface GigabitEthernet 0/0
Router(config-if)# no shutdown
Router(config-if)# exit

! Subinterface for VLAN 10
Router(config)# interface GigabitEthernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit

! Subinterface for VLAN 20
Router(config)# interface GigabitEthernet 0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit
```

---

## 9. Verifying VLAN Configuration

```ios
! Show all VLANs and port assignments
SW-CORE-01# show vlan brief

! Show detailed VLAN info
SW-CORE-01# show vlan id 10

! Show trunk ports
SW-CORE-01# show interfaces trunk

! Show a specific interface's VLAN assignment
SW-CORE-01# show interfaces FastEthernet 0/1 switchport

! Show all switchport info for all interfaces
SW-CORE-01# show interfaces switchport

! Show IP routing table (Layer 3 switch)
SW-CORE-01# show ip route
```

---

## 10. Deleting VLANs

```ios
! Delete a single VLAN
SW-CORE-01(config)# no vlan 40

! Delete all VLANs (factory reset VLANs)
SW-CORE-01# delete flash:vlan.dat
SW-CORE-01# reload
```

> ⚠️ **Warning:** Deleting a VLAN while ports are still assigned to it leaves those ports in an inactive state — devices will lose connectivity. Reassign ports first.

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Devices on same VLAN can't communicate | Port not assigned to correct VLAN | Check `show vlan brief` and `show interfaces switchport` |
| Trunk not passing VLAN traffic | VLAN not in allowed list | Add VLAN to trunk: `switchport trunk allowed vlan add X` |
| VLAN not in database | VLAN deleted or never created | Recreate VLAN: `vlan X` → `name NAME` |
| Management VLAN unreachable | Wrong IP or SVI shut down | Check `show interface vlan 99` — ensure it's up |
| Inter-VLAN routing not working | IP routing not enabled | Run `ip routing` on Layer 3 switch |
| Native VLAN mismatch warning | Both sides have different native VLANs | Set same native VLAN on both ends of the trunk |

---

## Quick Reference

```ios
! Create VLAN
vlan 10
name IT-Department

! Assign port to VLAN
interface FastEthernet 0/1
switchport mode access
switchport access vlan 10

! Configure trunk
interface GigabitEthernet 0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99

! Show VLANs
show vlan brief

! Show trunks
show interfaces trunk

! Show port VLAN info
show interfaces FastEthernet 0/1 switchport

! Enable inter-VLAN routing (Layer 3 switch)
ip routing
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown
```

---

## Notes
- Never use VLAN 1 for user traffic — it is the default and receives all untagged traffic, making it a security risk
- Always explicitly configure ports as access or trunk — don't rely on DTP auto-negotiation
- When creating VLANs ensure they exist on ALL switches in the path for traffic to flow
- VLAN 1002–1005 are reserved for Token Ring and FDDI — do not use them
- The native VLAN should match on both ends of a trunk link or traffic issues will occur

---

## Related Documents
- [Cisco Switch Configuration](cisco-switch-configuration.md)
- [Cisco Router Configuration](cisco-router-configuration.md)
- [SNMP Configuration](snmp-configuration-cisco-switch.md)
- [Cisco Switch Troubleshooting](../Hardware/cisco-switch-troubleshooting.md)
