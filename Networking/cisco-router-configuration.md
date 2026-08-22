# Cisco Router Configuration
> A guide to initial setup, routing configuration, and management of Cisco routers using IOS CLI.

**Category:** Networking  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Router** | Router | A Layer 3 device that forwards packets between different networks |
| **Interface** | Interface | A physical or virtual port on a router connected to a network |
| **IP Routing** | IP Routing | The process of forwarding packets based on destination IP address |
| **Routing Table** | Routing Table | A list of known networks and how to reach them |
| **Static Route** | Static Route | A manually configured route to a specific network |
| **Default Route** | Default Route | A catch-all route used when no specific route exists (0.0.0.0/0) |
| **Dynamic Routing** | Dynamic Routing | Routes automatically learned from other routers using a protocol |
| **OSPF** | Open Shortest Path First | A common dynamic routing protocol used in enterprise networks |
| **EIGRP** | Enhanced Interior Gateway Routing Protocol | Cisco's proprietary dynamic routing protocol |
| **NAT** | Network Address Translation | Translates private IP addresses to a public IP for internet access |
| **PAT** | Port Address Translation | Many-to-one NAT — multiple devices share one public IP |
| **ACL** | Access Control List | A set of rules permitting or denying traffic |
| **WAN** | Wide Area Network | The external network — typically the internet or an ISP connection |
| **LAN** | Local Area Network | The internal network |
| **Subinterface** | Subinterface | A virtual interface on a physical port — used for router-on-a-stick VLAN routing |
| **GRE** | Generic Routing Encapsulation | A tunneling protocol used to create VPNs |

---

## Overview
Routers operate at Layer 3 of the OSI model. While switches connect devices within a network using MAC addresses, routers connect different networks together using IP addresses.

**What routers do:**
- Connect your LAN to the internet (WAN)
- Route traffic between different VLANs or subnets
- Provide NAT to share a single public IP among many devices
- Apply security policies via ACLs
- Create VPN tunnels between sites

---

## Prerequisites
- Cisco router (ISR 1900, 2900, 4000 series, etc.)
- Console cable and terminal emulator
- Basic Cisco CLI knowledge (see Cisco Switch Configuration)

---

## 1. Initial Setup

```ios
! Enter privileged mode
Router> enable
Router#

! Enter config mode
Router# configure terminal

! Set hostname
Router(config)# hostname RTR-MAIN-01

! Set enable secret
RTR-MAIN-01(config)# enable secret StrongPassword123

! Disable DNS lookup
RTR-MAIN-01(config)# no ip domain-lookup

! Set MOTD banner
RTR-MAIN-01(config)# banner motd #
Authorized Access Only
#

! Encrypt all passwords
RTR-MAIN-01(config)# service password-encryption

! Save config
RTR-MAIN-01# wr
```

---

## 2. Configuring Interfaces

Routers have physical interfaces (GigabitEthernet, Serial) and virtual subinterfaces. All interfaces are **shutdown by default** — you must manually enable them.

```ios
! Configure LAN interface
RTR-MAIN-01(config)# interface GigabitEthernet 0/0
RTR-MAIN-01(config-if)# description LAN - Office Network
RTR-MAIN-01(config-if)# ip address 192.168.1.1 255.255.255.0
RTR-MAIN-01(config-if)# no shutdown
RTR-MAIN-01(config-if)# exit

! Configure WAN interface (internet)
RTR-MAIN-01(config)# interface GigabitEthernet 0/1
RTR-MAIN-01(config-if)# description WAN - ISP Connection
RTR-MAIN-01(config-if)# ip address 203.0.113.2 255.255.255.252
RTR-MAIN-01(config-if)# no shutdown
RTR-MAIN-01(config-if)# exit

! Verify interfaces
RTR-MAIN-01# show ip interface brief
RTR-MAIN-01# show interfaces GigabitEthernet 0/0
```

---

## 3. Configuring Static Routes

Static routes manually tell the router how to reach a specific network.

```ios
! Static route syntax:
! ip route [destination network] [subnet mask] [next-hop IP or exit interface]

! Route to a specific network via next-hop IP
RTR-MAIN-01(config)# ip route 192.168.2.0 255.255.255.0 192.168.1.254

! Default route (gateway of last resort — for internet access)
RTR-MAIN-01(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1

! Static route via exit interface
RTR-MAIN-01(config)# ip route 10.0.0.0 255.0.0.0 GigabitEthernet 0/1

! Remove a static route
RTR-MAIN-01(config)# no ip route 192.168.2.0 255.255.255.0 192.168.1.254

! Verify routing table
RTR-MAIN-01# show ip route
```

**Reading the routing table:**
```
C  192.168.1.0/24 is directly connected, GigabitEthernet0/0  ← C = Connected
S  192.168.2.0/24 [1/0] via 192.168.1.254                   ← S = Static
O  10.0.0.0/8 [110/2] via 192.168.1.254                     ← O = OSPF
S* 0.0.0.0/0 [1/0] via 203.0.113.1                          ← S* = Default route
```

---

## 4. Configuring NAT/PAT

NAT allows multiple devices with private IPs to share a single public IP address for internet access.

```ios
! Define the inside (LAN) interface
RTR-MAIN-01(config)# interface GigabitEthernet 0/0
RTR-MAIN-01(config-if)# ip nat inside
RTR-MAIN-01(config-if)# exit

! Define the outside (WAN/Internet) interface
RTR-MAIN-01(config)# interface GigabitEthernet 0/1
RTR-MAIN-01(config-if)# ip nat outside
RTR-MAIN-01(config-if)# exit

! Create an ACL defining which IPs to NAT
RTR-MAIN-01(config)# access-list 1 permit 192.168.1.0 0.0.0.255

! Configure PAT (overload) — many devices share one public IP
RTR-MAIN-01(config)# ip nat inside source list 1 interface GigabitEthernet 0/1 overload

! Verify NAT translations
RTR-MAIN-01# show ip nat translations
RTR-MAIN-01# show ip nat statistics

! Clear NAT table
RTR-MAIN-01# clear ip nat translation *
```

---

## 5. Configuring OSPF

OSPF is a dynamic routing protocol that automatically learns routes from other OSPF routers.

```ios
! Enable OSPF with process ID 1
RTR-MAIN-01(config)# router ospf 1

! Set router ID (recommended — use a unique loopback IP)
RTR-MAIN-01(config-router)# router-id 1.1.1.1

! Advertise networks into OSPF
! Syntax: network [network address] [wildcard mask] area [area ID]
RTR-MAIN-01(config-router)# network 192.168.1.0 0.0.0.255 area 0
RTR-MAIN-01(config-router)# network 10.0.0.0 0.0.0.3 area 0
RTR-MAIN-01(config-router)# exit

! Set passive interface (don't send OSPF hellos out this interface)
RTR-MAIN-01(config)# router ospf 1
RTR-MAIN-01(config-router)# passive-interface GigabitEthernet 0/0
RTR-MAIN-01(config-router)# exit

! Verify OSPF
RTR-MAIN-01# show ip ospf neighbor
RTR-MAIN-01# show ip ospf interface
RTR-MAIN-01# show ip route ospf
```

---

## 6. Configuring Access Control Lists (ACLs)

ACLs filter traffic based on source/destination IP and protocol.

```ios
! Standard ACL — filters by source IP only
RTR-MAIN-01(config)# access-list 10 permit 192.168.1.0 0.0.0.255
RTR-MAIN-01(config)# access-list 10 deny any

! Extended ACL — filters by source, destination, port, and protocol
RTR-MAIN-01(config)# access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 80
RTR-MAIN-01(config)# access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 443
RTR-MAIN-01(config)# access-list 100 deny ip any any

! Named ACL (easier to read and edit)
RTR-MAIN-01(config)# ip access-list extended BLOCK-TELNET
RTR-MAIN-01(config-ext-nacl)# deny tcp any any eq 23
RTR-MAIN-01(config-ext-nacl)# permit ip any any
RTR-MAIN-01(config-ext-nacl)# exit

! Apply ACL to an interface
RTR-MAIN-01(config)# interface GigabitEthernet 0/1
RTR-MAIN-01(config-if)# ip access-group 100 in    ! inbound traffic
RTR-MAIN-01(config-if)# ip access-group 100 out   ! outbound traffic
RTR-MAIN-01(config-if)# exit

! Verify ACLs
RTR-MAIN-01# show access-lists
RTR-MAIN-01# show ip interface GigabitEthernet 0/1
```

---

## 7. Configuring Router-on-a-Stick (Inter-VLAN Routing)

Uses subinterfaces on a single physical port to route between VLANs.

```ios
! Enable the physical interface
RTR-MAIN-01(config)# interface GigabitEthernet 0/0
RTR-MAIN-01(config-if)# no shutdown
RTR-MAIN-01(config-if)# exit

! Subinterface for VLAN 10
RTR-MAIN-01(config)# interface GigabitEthernet 0/0.10
RTR-MAIN-01(config-subif)# encapsulation dot1Q 10
RTR-MAIN-01(config-subif)# ip address 192.168.10.1 255.255.255.0
RTR-MAIN-01(config-subif)# exit

! Subinterface for VLAN 20
RTR-MAIN-01(config)# interface GigabitEthernet 0/0.20
RTR-MAIN-01(config-subif)# encapsulation dot1Q 20
RTR-MAIN-01(config-subif)# ip address 192.168.20.1 255.255.255.0
RTR-MAIN-01(config-subif)# exit

! Subinterface for native VLAN 99
RTR-MAIN-01(config)# interface GigabitEthernet 0/0.99
RTR-MAIN-01(config-subif)# encapsulation dot1Q 99 native
RTR-MAIN-01(config-subif)# ip address 192.168.99.1 255.255.255.0
RTR-MAIN-01(config-subif)# exit
```

---

## 8. SSH Configuration

```ios
! Set domain name
RTR-MAIN-01(config)# ip domain-name company.com

! Create local user
RTR-MAIN-01(config)# username admin privilege 15 secret AdminPass123

! Generate RSA keys
RTR-MAIN-01(config)# crypto key generate rsa modulus 2048

! Enable SSH v2
RTR-MAIN-01(config)# ip ssh version 2

! Configure VTY for SSH only
RTR-MAIN-01(config)# line vty 0 4
RTR-MAIN-01(config-line)# transport input ssh
RTR-MAIN-01(config-line)# login local
RTR-MAIN-01(config-line)# exit
```

---

## 9. Verifying and Troubleshooting

```ios
! View routing table
RTR-MAIN-01# show ip route

! View interface status
RTR-MAIN-01# show ip interface brief

! View detailed interface info
RTR-MAIN-01# show interfaces GigabitEthernet 0/0

! View NAT translations
RTR-MAIN-01# show ip nat translations

! View OSPF neighbors
RTR-MAIN-01# show ip ospf neighbor

! View ACLs and hit counts
RTR-MAIN-01# show access-lists

! Ping
RTR-MAIN-01# ping 8.8.8.8

! Traceroute
RTR-MAIN-01# traceroute 8.8.8.8

! Debug routing (use carefully — high CPU impact)
RTR-MAIN-01# debug ip routing
RTR-MAIN-01# undebug all

! View running config
RTR-MAIN-01# show running-config
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| No internet access | Missing default route or NAT misconfigured | Check `show ip route` for default route, verify NAT config |
| Can't ping between subnets | Missing routes or wrong next-hop | Check routing table, verify static routes |
| Interface shows down/down | Cable issue or no shutdown not run | Check cable, run `no shutdown` on interface |
| OSPF neighbors not forming | Mismatched area, timers, or MTU | Check `show ip ospf neighbor`, verify area IDs match |
| ACL blocking wrong traffic | ACL logic error | Check `show access-lists` hit counts, review permit/deny order |
| SSH not working | RSA keys not generated or wrong VTY config | Regenerate RSA keys, check VTY transport input ssh |

---

## Quick Reference

```ios
! View routing table
show ip route

! View interface summary
show ip interface brief

! Configure interface
interface GigabitEthernet 0/0
ip address 192.168.1.1 255.255.255.0
no shutdown

! Add default route
ip route 0.0.0.0 0.0.0.0 [next-hop-IP]

! Add static route
ip route [network] [mask] [next-hop]

! Enable NAT
ip nat inside source list 1 interface GigabitEthernet 0/1 overload

! Enable OSPF
router ospf 1
network 192.168.1.0 0.0.0.255 area 0

! Save config
wr
```

---

## Notes
- All router interfaces are shutdown by default — always run `no shutdown`
- The default route `0.0.0.0 0.0.0.0` is your gateway to the internet — without it nothing routes out
- NAT `overload` (PAT) is the most common configuration for small to medium offices
- ACL rules are processed top to bottom — first match wins — there is an implicit deny all at the end
- Always save config after changes — `wr` or `copy run start`

---

## Related Documents
- [Cisco Switch Configuration](cisco-switch-configuration.md)
- [VLAN Configuration](vlan-configuration-cisco-switch.md)
- [VPN Installation](vpn-installation.md)
- [Cisco Router Troubleshooting](../Hardware/cisco-router-troubleshooting.md)
