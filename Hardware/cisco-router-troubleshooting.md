# Cisco Router Troubleshooting
> A guide to diagnosing and resolving common issues on Cisco routers.

**Category:** Hardware  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Routing Table** | Routing Table | The list of known networks and how to reach them |
| **Default Route** | Default Route | The catch-all route used when no specific route matches (0.0.0.0/0) |
| **Next Hop** | Next Hop | The next router IP address packets are forwarded to |
| **Administrative Distance** | AD | A rating of route trustworthiness - lower is preferred |
| **NAT** | Network Address Translation | Translates private IPs to public IPs |
| **PAT** | Port Address Translation | Many-to-one NAT sharing one public IP |
| **ACL** | Access Control List | Rules permitting or denying traffic |
| **OSPF** | Open Shortest Path First | A dynamic routing protocol |
| **Adjacency** | OSPF Adjacency | A relationship formed between OSPF routers to exchange routing info |
| **CEF** | Cisco Express Forwarding | A fast packet switching mechanism |
| **ARP** | Address Resolution Protocol | Maps IP addresses to MAC addresses |
| **MTU** | Maximum Transmission Unit | The largest packet size a link can carry |
| **Flapping** | Interface Flapping | An interface repeatedly going up and down |

---

## Overview
Router troubleshooting follows the OSI model - start at Layer 1 (physical) and work up to Layer 3 (routing). Most issues fall into:
- **Physical** - cables, interface status, power
- **Data Link** - encapsulation, clocking (serial links)
- **Network** - routing table, NAT, ACLs
- **Configuration** - missing or wrong settings

---

## 1. Interface Not Coming Up

```ios
! Check all interface statuses
RTR-MAIN-01# show ip interface brief
! Look for line/protocol status:
! up/up     = working correctly
! up/down   = Layer 1 ok, Layer 2 problem
! down/down = Layer 1 problem (cable, no signal)
! admin down/down = manually shut down

! Check detailed interface info
RTR-MAIN-01# show interfaces GigabitEthernet 0/0
! Look for: input errors, CRC, resets, carrier transitions

! Bring up a shut down interface
RTR-MAIN-01(config)# interface GigabitEthernet 0/0
RTR-MAIN-01(config-if)# no shutdown

! Check interface error counters
RTR-MAIN-01# show interfaces GigabitEthernet 0/0 | include error|reset|CRC
```

### Interface Status Meanings

| Line / Protocol | Meaning | Common Cause |
|----------------|---------|-------------|
| up / up | Working | - |
| up / down | Physical OK, Layer 2 problem | Encapsulation mismatch, keepalive issue |
| down / down | No physical signal | Cable unplugged, device off, wrong cable |
| admin down / down | Manually disabled | Run `no shutdown` |

---

## 2. No Internet / Routing Issues

```ios
! Check the routing table
RTR-MAIN-01# show ip route
! Look for:
! S* 0.0.0.0/0 - default route (needed for internet)
! C  = connected network
! S  = static route
! O  = OSPF route

! Check if default route exists
RTR-MAIN-01# show ip route 0.0.0.0

! Check if a specific destination is reachable
RTR-MAIN-01# show ip route 8.8.8.8

! Test connectivity
RTR-MAIN-01# ping 8.8.8.8
RTR-MAIN-01# ping 192.168.1.1

! Trace the path
RTR-MAIN-01# traceroute 8.8.8.8

! Check ARP table - can the router reach the next hop?
RTR-MAIN-01# show arp
```

### Common Routing Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| No internet | Missing default route | `ip route 0.0.0.0 0.0.0.0 [ISP-IP]` |
| Can't reach remote subnet | Missing static route | Add route: `ip route [network] [mask] [next-hop]` |
| Route in table but not working | Wrong next hop or interface | Verify next-hop IP is reachable |
| Routes disappearing | Dynamic routing issue or interface flapping | Check OSPF adjacency, fix flapping interface |

---

## 3. NAT Troubleshooting

```ios
! Check NAT translations (active sessions)
RTR-MAIN-01# show ip nat translations

! Check NAT statistics
RTR-MAIN-01# show ip nat statistics
! Look for: hits (successful translations), misses (failed)

! Check NAT configuration
RTR-MAIN-01# show running-config | include nat

! Verify interfaces are marked inside/outside
RTR-MAIN-01# show ip interface GigabitEthernet 0/0
! Look for: Inbound/Outbound access list, NAT status

! Clear NAT translation table
RTR-MAIN-01# clear ip nat translation *

! Debug NAT (use carefully - verbose output)
RTR-MAIN-01# debug ip nat
RTR-MAIN-01# undebug all
```

### Common NAT Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| No internet through NAT | Inside/outside not set on interfaces | Add `ip nat inside` / `ip nat outside` to interfaces |
| Some devices not getting NAT | ACL too restrictive | Check the ACL used in the NAT statement |
| NAT translations not clearing | Long timeout values | `clear ip nat translation *` |
| Asymmetric routing breaking NAT | Traffic returning via different path | Ensure traffic enters and exits same interface |

```ios
! Verify NAT interface configuration
RTR-MAIN-01# show ip interface GigabitEthernet 0/0 | include NAT

! Check the ACL used for NAT
RTR-MAIN-01# show access-lists
```

---

## 4. ACL Troubleshooting

```ios
! View all ACLs and hit counts
RTR-MAIN-01# show access-lists

! View a specific ACL
RTR-MAIN-01# show access-lists 100

! Check which ACLs are applied to which interfaces
RTR-MAIN-01# show ip interface GigabitEthernet 0/1
! Look for: Inbound access list, Outbound access list

! Clear ACL hit counters
RTR-MAIN-01# clear access-list counters

! Debug ACL matches (use carefully)
RTR-MAIN-01# debug ip packet detail
RTR-MAIN-01# undebug all
```

### ACL Troubleshooting Logic
```
Traffic being blocked unexpectedly?
1. show access-lists -> check hit counts -> which rule is matching?
2. ACL rules are processed top to bottom - first match wins
3. There is an implicit deny all at the end - if nothing matches, traffic is denied
4. Check both inbound and outbound ACLs on the interface
5. Standard ACLs should be placed close to the destination
6. Extended ACLs should be placed close to the source
```

---

## 5. OSPF Troubleshooting

```ios
! Check OSPF neighbor relationships
RTR-MAIN-01# show ip ospf neighbor
! State should be FULL - anything else indicates a problem

! OSPF neighbor states:
! FULL     = working correctly
! 2WAY     = DR/BDR election - normal on multi-access networks
! EXSTART  = negotiating master/slave - should progress quickly
! EXCHANGE = exchanging DBD packets
! LOADING  = exchanging LSAs
! INIT     = received hello but not two-way yet
! DOWN     = no hellos received

! Check OSPF configuration
RTR-MAIN-01# show ip ospf
RTR-MAIN-01# show ip ospf interface

! Check OSPF routes in routing table
RTR-MAIN-01# show ip route ospf

! Verify OSPF network statements
RTR-MAIN-01# show running-config | section router ospf

! Debug OSPF adjacency formation
RTR-MAIN-01# debug ip ospf adj
RTR-MAIN-01# undebug all
```

### Common OSPF Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Neighbors not forming | Mismatched area ID | Verify both routers use the same area |
| Neighbors not forming | Mismatched hello/dead timers | Match timers on both sides |
| Neighbors not forming | Mismatched authentication | Match OSPF authentication config |
| Neighbors not forming | MTU mismatch | Match MTU or disable MTU checking |
| Route not in routing table | Network not advertised | Check `network` statement in OSPF config |
| Route flapping | Unstable interface | Fix the physical issue causing flapping |

---

## 6. Interface Flapping

Interface flapping (repeatedly going up/down) causes routing instability and service disruption.

```ios
! Check interface for carrier transitions
RTR-MAIN-01# show interfaces GigabitEthernet 0/0
! Look for: X input resets, X carrier transitions - high numbers indicate flapping

! View interface history
RTR-MAIN-01# show interfaces GigabitEthernet 0/0 | include transition|reset|flap

! View system log for interface changes
RTR-MAIN-01# show logging | include GigabitEthernet|UPDOWN|CHANGED

! Dampen interface flapping (delays bringing interface back up)
RTR-MAIN-01(config)# interface GigabitEthernet 0/0
RTR-MAIN-01(config-if)# dampening
```

**Common causes of flapping:**
- Faulty cable - replace the cable
- Faulty SFP - replace the SFP transceiver
- Speed/duplex mismatch - set manually on both ends
- PoE power issue (if applicable)
- Remote device NIC issues

---

## 7. High CPU Usage

```ios
! Check CPU utilization
RTR-MAIN-01# show processes cpu
RTR-MAIN-01# show processes cpu sorted
! Look for processes consuming high CPU

! Check for process causing spike
RTR-MAIN-01# show processes cpu history

! Common high CPU causes:
! - IP Input process high -> routing loop or broadcast storm
! - Spanning Tree -> STP topology change
! - BGP/OSPF -> routing instability
! - CEF process -> CEF switching issue
```

---

## 8. Memory Issues

```ios
! Check memory usage
RTR-MAIN-01# show memory
RTR-MAIN-01# show memory statistics

! Check memory allocations
RTR-MAIN-01# show memory summary

! Check for memory leaks
RTR-MAIN-01# show processes memory sorted
```

---

## 9. Connectivity Testing

```ios
! Basic ping
RTR-MAIN-01# ping 8.8.8.8

! Extended ping - test from specific source interface
RTR-MAIN-01# ping 8.8.8.8 source GigabitEthernet 0/0
RTR-MAIN-01# ping 8.8.8.8 repeat 100 size 1500

! Traceroute
RTR-MAIN-01# traceroute 8.8.8.8

! DNS lookup
RTR-MAIN-01# ping google.com

! Test specific TCP port
RTR-MAIN-01# telnet 8.8.8.8 53

! Check ARP for next hop
RTR-MAIN-01# show arp
RTR-MAIN-01# clear arp-cache
```

---

## 10. General Diagnostic Commands

```ios
! View full running config
RTR-MAIN-01# show running-config

! View system info and uptime
RTR-MAIN-01# show version

! View system logs
RTR-MAIN-01# show logging

! View interface summary
RTR-MAIN-01# show ip interface brief

! View routing table
RTR-MAIN-01# show ip route

! View all NAT translations
RTR-MAIN-01# show ip nat translations

! View all ACLs
RTR-MAIN-01# show access-lists

! View CDP neighbors
RTR-MAIN-01# show cdp neighbors detail

! View OSPF neighbors
RTR-MAIN-01# show ip ospf neighbor

! Check flash and IOS
RTR-MAIN-01# show flash
RTR-MAIN-01# show version | include IOS
```

---

## Common Issues & Quick Fix Reference

| Problem | First Command | Likely Fix |
|---------|--------------|------------|
| No internet | `show ip route` | Add default route |
| Interface down | `show ip interface brief` | `no shutdown` on interface |
| NAT not working | `show ip nat translations` | Check inside/outside config |
| ACL blocking traffic | `show access-lists` | Review ACL rule order |
| OSPF not forming | `show ip ospf neighbor` | Check area, timers, authentication |
| Routing loop | `traceroute` | Check routing table for recursive routes |
| High CPU | `show processes cpu sorted` | Identify and fix causing process |

---

## Notes
- Always `undebug all` after using any debug command - debug output consumes significant CPU
- The routing table (`show ip route`) is the most important command for Layer 3 troubleshooting
- ACL hit counters show you exactly which rule is matching - use this to debug unexpected blocks
- OSPF neighbor state must be FULL for routes to be exchanged - any other state needs investigation
- Always save config after changes - `wr`

---

## Related Documents
- [Cisco Router Configuration](cisco-router-configuration.md)
- [Cisco Switch Troubleshooting](cisco-switch-troubleshooting.md)
- [VPN Installation](vpn-installation.md)
