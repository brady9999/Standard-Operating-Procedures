# Cisco Switch Troubleshooting
> A guide to diagnosing and resolving common issues on Cisco Catalyst switches.

**Category:** Hardware  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **IOS** | Internetwork Operating System | Cisco's operating system running on switches and routers |
| **Port** | Switch Port | A physical interface on the switch |
| **LED** | Light Emitting Diode | Status indicators on the switch front panel |
| **VLAN** | Virtual LAN | A logical network segment |
| **Trunk** | Trunk Port | A port carrying multiple VLANs |
| **STP** | Spanning Tree Protocol | Prevents network loops |
| **CDP** | Cisco Discovery Protocol | Discovers neighboring Cisco devices |
| **MAC Table** | MAC Address Table | Maps MAC addresses to switch ports |
| **BPDU** | Bridge Protocol Data Unit | STP messages exchanged between switches |
| **PortFast** | PortFast | Bypasses STP listening/learning for access ports |
| **BPDU Guard** | BPDU Guard | Shuts down a PortFast port if a BPDU is received |
| **ErrDisabled** | Error Disabled | A port state where the switch has disabled the port due to an error condition |
| **Duplex** | Duplex | Whether a port communicates in one direction at a time (half) or both simultaneously (full) |
| **Auto-negotiation** | Auto-negotiation | Automatic agreement between two devices on speed and duplex |

---

## Overview
Cisco switch troubleshooting follows a layered approach — start at the physical layer and work up. Most issues fall into:
- **Physical** — cables, ports, LEDs, power
- **Data Link** — VLANs, trunks, STP, MAC table
- **Network** — IP addressing, routing between VLANs
- **Configuration** — wrong settings, missing config

---

## Switch LED Status Guide

### System LED
| Color | Meaning |
|-------|---------|
| Green | Normal operation |
| Amber | POST failure or fault |
| Off | Not powered |

### Port LEDs
| Color/State | Meaning |
|------------|---------|
| Off | No link — no cable or device |
| Green | Link established |
| Blinking Green | Activity — traffic passing |
| Amber | Port disabled or STP blocking |
| Alternating Green/Amber | Link fault |

### Mode Button
Press the **Mode** button to cycle through LED modes:
- **STAT** — port status (default)
- **DPLX** — duplex (green = full, amber = half)
- **SPEED** — port speed

---

## 1. Port Not Coming Up (Link Down)

### Step by Step
```ios
! Check interface status
SW-CORE-01# show interfaces FastEthernet 0/1
! Look for: line protocol is down, input errors, CRC

! Check interface status table
SW-CORE-01# show interfaces status
! Look for: connected, notconnect, disabled, err-disabled

! Check if port is administratively shut down
SW-CORE-01# show running-config interface FastEthernet 0/1
! If you see "shutdown" — the port is manually disabled

! Bring the port up
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# no shutdown
```

### Common Causes
| Symptom | Cause | Fix |
|---------|-------|-----|
| Port LED off | No cable, bad cable, or dead device | Test cable, try different port |
| Port shows notconnect | Device not sending link signal | Check device NIC, try different cable |
| Port shows disabled | Manually shut down | `no shutdown` on the interface |
| Port shows err-disabled | Error condition triggered shutdown | See ErrDisabled section below |
| Port amber | STP blocking | Normal if redundant link — check STP |

---

## 2. ErrDisabled Port Recovery

ErrDisabled means the switch automatically shut down the port due to a policy violation or error.

```ios
! Check why the port went err-disabled
SW-CORE-01# show interfaces status err-disabled
SW-CORE-01# show errdisable recovery

! Common causes shown in the output:
! - psecure-violation  → Port security MAC violation
! - bpduguard         → BPDU received on PortFast port
! - channel-misconfig → EtherChannel misconfiguration
! - link-flap         → Port was flapping up/down too fast

! Manually recover the port
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# shutdown
SW-CORE-01(config-if)# no shutdown

! Enable automatic recovery for specific causes
SW-CORE-01(config)# errdisable recovery cause psecure-violation
SW-CORE-01(config)# errdisable recovery interval 300
```

---

## 3. VLAN Issues

```ios
! Check VLAN database — is the VLAN created?
SW-CORE-01# show vlan brief

! Check which VLAN a port is assigned to
SW-CORE-01# show interfaces FastEthernet 0/1 switchport
! Look for: Access Mode VLAN and Operational Mode

! Check trunk ports — which VLANs are allowed?
SW-CORE-01# show interfaces trunk
! Look for: VLANs allowed and active in management domain

! Check if VLAN is active
SW-CORE-01# show vlan id 10
! Status should show "active" not "act/lshut"
```

### Common VLAN Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Device can't reach others on same VLAN | VLAN not created or port in wrong VLAN | Check `show vlan brief`, assign correct VLAN |
| VLAN traffic not crossing trunk | VLAN not in allowed list | `switchport trunk allowed vlan add X` |
| VLAN shows as inactive | VLAN deleted or not created on this switch | Recreate VLAN: `vlan X` → `name NAME` |
| Native VLAN mismatch warning | Both sides have different native VLANs | Match native VLAN on both trunk ends |

---

## 4. Spanning Tree Issues

```ios
! Check STP status for all VLANs
SW-CORE-01# show spanning-tree

! Check STP for a specific VLAN
SW-CORE-01# show spanning-tree vlan 10

! Check which ports are blocking
SW-CORE-01# show spanning-tree detail
! Look for: BLK = blocking, FWD = forwarding, LIS = listening, LRN = learning

! Find the root bridge
SW-CORE-01# show spanning-tree | include Root

! Check STP on a specific port
SW-CORE-01# show spanning-tree interface FastEthernet 0/1
```

### Common STP Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Slow port coming up (30-50 seconds) | STP going through listening/learning | Enable PortFast on access ports |
| Network loop causing broadcast storm | STP not blocking a redundant link | Check STP topology, verify BPDU exchange |
| Port stuck in BLK state unexpectedly | STP blocking legitimate link | Check root bridge placement, adjust priorities |
| BPDU Guard shutting down port | Unauthorized switch connected | Remove unauthorized switch or disable BPDU Guard on that port |

```ios
! Enable PortFast on access ports (skips STP listening/learning)
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# spanning-tree portfast

! Enable BPDU Guard on PortFast ports
SW-CORE-01(config-if)# spanning-tree bpduguard enable

! Set switch as root bridge for a VLAN
SW-CORE-01(config)# spanning-tree vlan 10 root primary
```

---

## 5. MAC Address Table Issues

```ios
! View the full MAC address table
SW-CORE-01# show mac address-table

! Find which port a specific MAC is on
SW-CORE-01# show mac address-table address 0011.2233.4455

! Find all MACs on a specific port
SW-CORE-01# show mac address-table interface FastEthernet 0/1

! Find all MACs on a specific VLAN
SW-CORE-01# show mac address-table vlan 10

! Clear the dynamic MAC table
SW-CORE-01# clear mac address-table dynamic

! Check MAC table size
SW-CORE-01# show mac address-table count
```

---

## 6. Speed and Duplex Issues

Speed/duplex mismatch causes poor performance, errors, and retransmissions even when the link appears up.

```ios
! Check speed and duplex on an interface
SW-CORE-01# show interfaces FastEthernet 0/1
! Look for: Full-duplex, 100Mb/s — or Half-duplex which indicates mismatch

! Check all interfaces for duplex issues
SW-CORE-01# show interfaces status
! Look for: a-100 (auto-negotiated 100Mb) vs 100 (manually set)

! Set speed and duplex manually (both ends must match)
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# speed 100
SW-CORE-01(config-if)# duplex full

! Return to auto-negotiation
SW-CORE-01(config-if)# speed auto
SW-CORE-01(config-if)# duplex auto
```

### Signs of Duplex Mismatch
- High input errors and CRC errors
- Slow network performance on a specific port
- Late collisions in the interface counters

```ios
! Check for errors indicating duplex mismatch
SW-CORE-01# show interfaces FastEthernet 0/1
! Look for: input errors, CRC, late collision — these indicate duplex mismatch
```

---

## 7. High CPU / Performance Issues

```ios
! Check CPU utilization
SW-CORE-01# show processes cpu
SW-CORE-01# show processes cpu sorted
! Look for processes consuming high CPU

! Check for broadcast storms
SW-CORE-01# show interfaces | include input rate
! High input rates on many ports = possible broadcast storm

! Check CDP for neighbors sending excessive traffic
SW-CORE-01# show cdp neighbors detail

! Check logging for errors
SW-CORE-01# show logging
```

---

## 8. Power over Ethernet (PoE) Issues

```ios
! Check PoE status on all ports
SW-CORE-01# show power inline

! Check PoE on a specific port
SW-CORE-01# show power inline FastEthernet 0/1

! Check total PoE budget and usage
SW-CORE-01# show power inline consumption

! Set PoE power limit on a port
SW-CORE-01(config)# interface FastEthernet 0/1
SW-CORE-01(config-if)# power inline consumption 15400
! 15400 milliwatts = 15.4W (PoE standard)

! Disable PoE on a port
SW-CORE-01(config-if)# power inline never

! Re-enable PoE on a port
SW-CORE-01(config-if)# power inline auto
```

---

## 9. Connectivity Troubleshooting

```ios
! Ping from switch
SW-CORE-01# ping 192.168.1.1

! Extended ping (more options)
SW-CORE-01# ping
! Then answer the prompts for source, repeat count, size

! Trace route
SW-CORE-01# traceroute 192.168.1.1

! Check ARP table
SW-CORE-01# show arp

! Check CDP neighbors — verify connected devices
SW-CORE-01# show cdp neighbors
SW-CORE-01# show cdp neighbors detail

! Check IP interface
SW-CORE-01# show ip interface brief
SW-CORE-01# show interface vlan 1
```

---

## 10. Configuration and Log Review

```ios
! View running configuration
SW-CORE-01# show running-config

! View specific interface config
SW-CORE-01# show running-config interface FastEthernet 0/1

! View system logs
SW-CORE-01# show logging

! View recent log messages
SW-CORE-01# show logging | include %

! Check version and hardware info
SW-CORE-01# show version

! Check flash storage
SW-CORE-01# show flash
```

---

## Quick Troubleshooting Decision Tree

```
Device can't communicate?
├── Check port LED → Off?
│   └── Cable issue → Try different cable/port
├── Port shows err-disabled?
│   └── show errdisable recovery → Fix cause → shutdown/no shutdown
├── Port up but no VLAN traffic?
│   └── show vlan brief → VLAN exists? → show interfaces trunk → VLAN allowed?
├── Port slow or errors?
│   └── show interfaces → CRC/collision errors? → Check duplex mismatch
├── Port takes 30s to come up?
│   └── Enable spanning-tree portfast on access port
└── Everything looks right but still broken?
    └── show mac address-table → Is the MAC being learned on the right port?
```

---

## Quick Reference Commands

```ios
! Most useful troubleshooting commands
show interfaces status          ! All ports — status, VLAN, speed, duplex
show interfaces FastEthernet 0/1 ! Detailed — errors, counters, state
show vlan brief                 ! All VLANs and port assignments
show interfaces trunk           ! Trunk ports and allowed VLANs
show spanning-tree              ! STP state for all VLANs
show mac address-table          ! MAC to port mappings
show power inline               ! PoE status
show errdisable recovery        ! Why ports went err-disabled
show cdp neighbors detail       ! Connected Cisco devices
show logging                    ! System log messages
show processes cpu              ! CPU utilization
ping 192.168.1.1                ! Connectivity test
```

---

## Notes
- Always start with `show interfaces status` — it gives the fastest overview of all port states
- ErrDisabled ports must be manually recovered with `shutdown` then `no shutdown` unless auto-recovery is configured
- STP is often the cause of slow port activation — use PortFast on all access ports
- Duplex mismatch is subtle — the link is up but performance is terrible and there are CRC errors
- Always save config after changes — `wr`

---

## Related Documents
- [Cisco Switch Configuration](cisco-switch-configuration.md)
- [VLAN Configuration](vlan-configuration-cisco-switch.md)
- [SNMP Configuration](snmp-configuration-cisco-switch.md)
