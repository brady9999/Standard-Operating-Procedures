# Cisco Switch Configuration
> A guide to initial setup, configuration, and management of Cisco Catalyst switches using CLI.

**Category:** Networking  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **IOS** | Internetwork Operating System | Cisco's operating system that runs on switches and routers |
| **CLI** | Command Line Interface | The text-based interface used to configure Cisco devices |
| **Console Port** | Console Port | A physical port used to connect directly to the device for initial setup |
| **SSH** | Secure Shell | Encrypted remote management protocol — preferred over Telnet |
| **Telnet** | Telnet | Unencrypted remote management — avoid in production |
| **VLAN** | Virtual Local Area Network | A logical segmentation of a network |
| **Trunk** | Trunk Port | A port that carries traffic for multiple VLANs |
| **Access Port** | Access Port | A port assigned to a single VLAN for end devices |
| **STP** | Spanning Tree Protocol | Prevents network loops by blocking redundant paths |
| **MAC Table** | MAC Address Table | A table mapping MAC addresses to switch ports |
| **Port Security** | Port Security | A feature limiting which devices can connect to a port |
| **MOTD** | Message of the Day | A login banner displayed when connecting to the device |
| **NVRAM** | Non-Volatile RAM | Where the startup configuration is stored |
| **Running Config** | Running Configuration | The active configuration currently in memory |
| **Startup Config** | Startup Configuration | The saved configuration loaded at boot |
| **Enable Mode** | Privileged EXEC Mode | The elevated mode for viewing and managing the device |
| **Config Mode** | Global Configuration Mode | The mode for making configuration changes |
| **Interface Mode** | Interface Configuration Mode | The mode for configuring specific ports |

---

## Overview
Cisco switches are the backbone of most enterprise networks. They connect devices together at Layer 2 (Data Link layer) using MAC addresses, and can be configured to segment traffic using VLANs, control access with port security, and provide redundancy with Spanning Tree Protocol.

**IOS Modes:**
```
User EXEC Mode        switch>           View basic info only
Privileged EXEC Mode  switch#           Full access to view config
Global Config Mode    switch(config)#   Make configuration changes
Interface Mode        switch(config-if)# Configure specific ports
```

---

## Prerequisites
- Cisco Catalyst switch (any model)
- Console cable (RJ45 to DB9 or USB-to-Serial adapter)
- Terminal emulator — PuTTY, SecureCRT, or minicom on Linux
- Console settings: **9600 baud, 8 data bits, no parity, 1 stop bit, no flow control**

---

## Connecting via Console

1. Connect console cable from PC to the switch's **Console** port
2. Open PuTTY → select **Serial** → set COM port → Speed `9600`
3. Click **Open**
4. Press **Enter** to get a prompt
5. You will see `Switch>` — you are in User EXEC mode

---

## 1. Initial Setup and Basic Configuration

```ios
! Enter privileged EXEC mode
Switch> enable
Switch#

! Enter global configuration mode
Switch# configure terminal
Switch(config)#

! Set the hostname
Switch(config)# hostname SW-CORE-01
SW-CORE-01(config)#

! Set the enable (privileged) password — encrypted
SW-CORE-01(config)# enable secret StrongPassword123

! Set a Message of the Day banner
SW-CORE-01(config)# banner motd #
Authorized Access Only. All activity is monitored.
#

! Disable DNS lookup (stops the switch trying to resolve typos as hostnames)
SW-CORE-01(config)# no ip domain-lookup

! Set the clock (if not using NTP)
SW-CORE-01# clock set 10:30:00 18 Aug 2026

! Save the configuration
SW-CORE-01# copy running-config startup-config
! Or shorthand:
SW-CORE-01# wr
```

---

## 2. Configuring Management IP Address

To manage the switch remotely it needs an IP address on VLAN 1 (or your management VLAN).

```ios
SW-CORE-01(config)# interface vlan 1
SW-CORE-01(config-if)# ip address 192.168.1.10 255.255.255.0
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

! Set the default gateway
SW-CORE-01(config)# ip default-gateway 192.168.1.1

! Verify
SW-CORE-01# show interface vlan 1
SW-CORE-01# ping 192.168.1.1
```

---

## 3. Configuring Passwords and SSH Access

### Setting Console and VTY Passwords
```ios
! Console line password
SW-CORE-01(config)# line console 0
SW-CORE-01(config-line)# password ConsolePass123
SW-CORE-01(config-line)# login
SW-CORE-01(config-line)# exit

! VTY lines (remote access — Telnet/SSH)
SW-CORE-01(config)# line vty 0 15
SW-CORE-01(config-line)# password VTYPass123
SW-CORE-01(config-line)# login
SW-CORE-01(config-line)# exit
```

### Enabling SSH (Recommended over Telnet)
```ios
! Set domain name (required for SSH)
SW-CORE-01(config)# ip domain-name company.com

! Create a local user
SW-CORE-01(config)# username admin privilege 15 secret AdminPass123

! Generate RSA keys (use 2048 for security)
SW-CORE-01(config)# crypto key generate rsa modulus 2048

! Set SSH version 2
SW-CORE-01(config)# ip ssh version 2

! Configure VTY to use SSH only and local authentication
SW-CORE-01(config)# line vty 0 15
SW-CORE-01(config-line)# transport input ssh
SW-CORE-01(config-line)# login local
SW-CORE-01(config-line)# exit

! Save
SW-CORE-01# wr
```

### Encrypting All Passwords in Config
```ios
SW-CORE-01(config)# service password-encryption
```

---

## 4. Configuring Access Ports

Access ports connect to end devices (computers, printers, phones) and carry traffic for a single VLAN.

```ios
! Configure a single port
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# description PC - Reception Desk
SW-CORE-01(config-if)# switchport mode access
SW-CORE-01(config-if)# switchport access vlan 10
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

! Configure a range of ports
SW-CORE-01(config)# interface range FastEthernet 0/1-24
SW-CORE-01(config-if-range)# switchport mode access
SW-CORE-01(config-if-range)# switchport access vlan 10
SW-CORE-01(config-if-range)# no shutdown
SW-CORE-01(config-if-range)# exit
```

---

## 5. Configuring Trunk Ports

Trunk ports carry traffic for multiple VLANs and connect switches to other switches or routers.

```ios
! Configure a trunk port
SW-CORE-01(config)# interface GigabitEthernet 0/1
SW-CORE-01(config-if)# description Uplink to Core Switch
SW-CORE-01(config-if)# switchport mode trunk
SW-CORE-01(config-if)# switchport trunk encapsulation dot1q
SW-CORE-01(config-if)# switchport trunk allowed vlan 10,20,30,99
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

! Verify trunk
SW-CORE-01# show interfaces trunk
```

---

## 6. Port Security

Port security limits which devices can connect to a port based on MAC address.

```ios
! Enable port security on a port
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# switchport mode access
SW-CORE-01(config-if)# switchport port-security

! Set maximum number of MAC addresses allowed (default is 1)
SW-CORE-01(config-if)# switchport port-security maximum 2

! Set violation action
! shutdown — disables the port (most common)
! restrict — drops packets and logs
! protect — drops packets silently
SW-CORE-01(config-if)# switchport port-security violation shutdown

! Sticky learning — automatically learns and saves MAC addresses
SW-CORE-01(config-if)# switchport port-security mac-address sticky

! Or manually specify a MAC address
SW-CORE-01(config-if)# switchport port-security mac-address 0011.2233.4455

SW-CORE-01(config-if)# exit

! View port security status
SW-CORE-01# show port-security
SW-CORE-01# show port-security interface FastEthernet 0/1

! Re-enable a port that was shut down by port security
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# shutdown
SW-CORE-01(config-if)# no shutdown
```

---

## 7. Disabling Unused Ports

Security best practice — shut down all ports not in use.

```ios
! Disable a range of unused ports
SW-CORE-01(config)# interface range FastEthernet 0/20-24
SW-CORE-01(config-if-range)# shutdown
SW-CORE-01(config-if-range)# description UNUSED - DISABLED
SW-CORE-01(config-if-range)# exit
```

---

## 8. Viewing and Verifying Configuration

```ios
! View running configuration
SW-CORE-01# show running-config

! View startup configuration
SW-CORE-01# show startup-config

! View all interfaces and their status
SW-CORE-01# show interfaces status

! View a specific interface
SW-CORE-01# show interfaces FastEthernet 0/1

! View VLAN assignments
SW-CORE-01# show vlan brief

! View MAC address table
SW-CORE-01# show mac address-table

! View spanning tree status
SW-CORE-01# show spanning-tree

! View CDP neighbors (connected Cisco devices)
SW-CORE-01# show cdp neighbors

! View IP interface brief (management IP status)
SW-CORE-01# show ip interface brief

! View ARP table
SW-CORE-01# show arp
```

---

## 9. Saving and Managing Configuration

```ios
! Save running config to startup config
SW-CORE-01# copy running-config startup-config

! Backup config to TFTP server
SW-CORE-01# copy running-config tftp://192.168.1.100/SW-CORE-01-backup.cfg

! Restore config from TFTP
SW-CORE-01# copy tftp://192.168.1.100/SW-CORE-01-backup.cfg running-config

! Erase startup config (factory reset)
SW-CORE-01# write erase
SW-CORE-01# reload
```

---

## 10. Common Troubleshooting Commands

```ios
! Check interface errors
SW-CORE-01# show interfaces FastEthernet 0/1
! Look for: input errors, output errors, CRC, collisions

! Check CDP neighbors — verify connected devices
SW-CORE-01# show cdp neighbors detail

! Ping from switch
SW-CORE-01# ping 192.168.1.1

! Trace route
SW-CORE-01# traceroute 192.168.1.1

! View log messages
SW-CORE-01# show logging

! Check port security violations
SW-CORE-01# show port-security

! Clear MAC address table
SW-CORE-01# clear mac address-table dynamic

! Check STP — find root bridge and blocked ports
SW-CORE-01# show spanning-tree detail
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Port not coming up | Cable issue or port disabled | Check cable, run `no shutdown` on the interface |
| Can't SSH to switch | SSH not configured or wrong IP | Verify IP, domain name, RSA keys, and VTY SSH config |
| VLAN traffic not passing | VLAN not created or trunk misconfigured | Check `show vlan brief` and `show interfaces trunk` |
| Port keeps shutting down | Port security violation | Check `show port-security`, clear violation, re-enable port |
| Can't save config | Flash memory full | Delete old files with `delete flash:filename` |
| Devices can't communicate | STP blocking the port | Check `show spanning-tree` for blocked ports |

---

## Quick Reference

```ios
! Enter enable mode
enable

! Enter config mode
configure terminal

! Save config
wr

! Show all interface statuses
show interfaces status

! Show VLANs
show vlan brief

! Show trunks
show interfaces trunk

! Show MAC table
show mac address-table

! Show CDP neighbors
show cdp neighbors

! Show running config
show running-config

! Ping
ping 192.168.1.1

! Set hostname
hostname SW-NAME

! Set management IP
interface vlan 1
ip address 192.168.1.10 255.255.255.0
no shutdown
```

---

## Notes
- Always save config with `wr` or `copy run start` after making changes — unsaved changes are lost on reboot
- Use `enable secret` not `enable password` — secret uses MD5 encryption
- SSH is always preferred over Telnet — Telnet sends passwords in plain text
- `no shutdown` is required to bring up interfaces — they default to shutdown on some models
- CDP (Cisco Discovery Protocol) reveals connected Cisco devices — disable on edge ports facing untrusted networks with `no cdp enable`

---

## Related Documents
- [VLAN Configuration](vlan-configuration-cisco-switch.md)
- [SNMP Configuration](snmp-configuration-cisco-switch.md)
- [Cisco Router Configuration](cisco-router-configuration.md)
- [Cisco Switch Troubleshooting](../Hardware/cisco-switch-troubleshooting.md)
