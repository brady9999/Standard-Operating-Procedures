# Cisco Access Point Troubleshooting
> A guide to diagnosing and resolving common issues with Cisco Aironet and Catalyst wireless access points.

**Category:** Hardware  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **AP** | Access Point | The wireless device clients connect to |
| **SSID** | Service Set Identifier | The wireless network name |
| **BSSID** | Basic Service Set Identifier | The MAC address of a specific AP radio |
| **WLC** | Wireless LAN Controller | A controller managing multiple APs centrally |
| **CAPWAP** | Control and Provisioning of Wireless Access Points | The protocol between AP and WLC |
| **PoE** | Power over Ethernet | Powering the AP through the ethernet cable |
| **RSSI** | Received Signal Strength Indicator | Signal strength - closer to 0 dBm is stronger |
| **SNR** | Signal-to-Noise Ratio | Signal quality - higher is better |
| **Channel** | Wireless Channel | A specific frequency within the band |
| **Band** | Frequency Band | 2.4 GHz or 5 GHz wireless frequencies |
| **Roaming** | Wireless Roaming | A client moving between APs while staying connected |
| **Deauth** | Deauthentication | Disconnecting a client from the wireless network |
| **Association** | Association | A client establishing a connection with an AP |
| **Authentication** | Wireless Authentication | Verifying the client's credentials to join the network |
| **TPC** | Transmit Power Control | Automatically adjusting AP transmit power |
| **DFS** | Dynamic Frequency Selection | Automatically avoiding radar frequencies on 5 GHz |

---

## Overview
Wireless troubleshooting is more complex than wired troubleshooting because the medium is invisible and shared. Issues can be caused by:
- **Power** - AP not receiving enough PoE power
- **Physical** - AP placement, obstructions, antenna issues
- **RF** - interference, channel overlap, signal strength
- **Configuration** - wrong SSID, VLAN, authentication settings
- **Client** - device-specific wireless driver or settings issues

---

## AP LED Status Guide

### Cisco Aironet LED States
| LED Color/Pattern | Meaning |
|------------------|---------|
| Off | No power |
| Blinking Green | Normal operation - booting or associated |
| Solid Green | Associated with WLC and operational |
| Blinking Amber | Discovery phase - looking for WLC |
| Solid Amber | Firmware loading or error |
| Alternating Red/Amber | Firmware upgrade in progress |
| Solid Red | Boot failure or critical error |
| Cycling Red/Green/Amber | Factory reset in progress |

---

## 1. AP Not Powering On

```
Physical checks:
1. Verify the ethernet cable is connected to a PoE-capable switch port
2. Check the switch port with: show power inline FastEthernet 0/X
3. Verify the switch has enough PoE budget remaining
4. Try a different ethernet cable
5. Try a different PoE switch port
6. Check if the AP has an optional power adapter and try that
```

```ios
! On the switch - check PoE status
SW-CORE-01# show power inline

! Check a specific port
SW-CORE-01# show power inline FastEthernet 0/12
! Look for: status = on, class = 3 or 4, watts consumed

! Check total PoE budget
SW-CORE-01# show power inline consumption

! If PoE not negotiating - reset the port
SW-CORE-01(config)# interface FastEthernet 0/12
SW-CORE-01(config-if)# shutdown
SW-CORE-01(config-if)# no shutdown
```

**PoE Power Classes:**
| Class | Max Power | Common Devices |
|-------|----------|----------------|
| 0 | 15.4W | Unknown device |
| 1 | 4W | VoIP phones |
| 2 | 7W | Basic APs |
| 3 | 15.4W | Standard APs |
| 4 | 30W | High-power APs (PoE+) |

---

## 2. AP Not Joining WLC (Lightweight AP)

Lightweight APs must join a WLC before they can operate. The discovery process uses several methods.

```ios
! On the WLC - check AP join status
(WLC)# show ap join stats summary all
(WLC)# show ap summary

! On the AP console - check CAPWAP status
AP# show capwap client rcb
AP# show capwap client config

! View CAPWAP discovery attempts
AP# debug capwap client events
AP# debug capwap client errors
```

### AP Discovery Order
```
1. DHCP Option 43     -> WLC IP sent with DHCP lease
2. DNS               -> AP resolves "CISCO-CAPWAP-CONTROLLER.domain"
3. Broadcast         -> AP broadcasts on local subnet
4. Primed WLC        -> WLC IP manually configured on AP
5. Previously joined -> AP remembers last WLC
```

### Configuring DHCP Option 43
```powershell
# On Windows DHCP Server - add Option 43 with WLC IP
# WLC IP 192.168.1.100 in hex = c0 a8 01 64
# Format for Option 43: f1:04:c0:a8:01:64

Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 `
  -OptionId 43 -Value "f104c0a80164"
```

### Firewall Ports Required for CAPWAP
```
UDP 5246 - CAPWAP control channel
UDP 5247 - CAPWAP data channel
```

### Common AP Join Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| AP stuck in discovery | Can't find WLC | Configure DHCP Option 43 or DNS |
| AP joins wrong WLC | Multiple WLCs on network | Configure AP to primary WLC |
| CAPWAP tunnel fails | Firewall blocking UDP 5246/5247 | Open firewall ports |
| AP joins but no clients | SSID not pushed from WLC | Check WLAN profile on WLC |
| Certificate error | Time mismatch between AP and WLC | Sync NTP on both devices |

---

## 3. Clients Can't Connect to SSID

### Client Can't See the SSID
```ios
! On autonomous AP - verify SSID is broadcast
AP# show dot11 bssid
! SSID should appear with guest-mode enabled

! Check radio interface is up
AP# show interfaces Dot11Radio0
! Should show: line protocol is up

! Verify SSID is applied to the radio
AP# show running-config | section Dot11Radio
```

### Client Sees SSID But Can't Connect
```
Troubleshooting steps:
1. Verify the correct password is being used
2. Check WPA version matches (WPA2 vs WPA3)
3. Forget the network on the client and reconnect
4. Check if MAC filtering is blocking the client
5. Verify the VLAN is correctly configured on the AP port
6. Check the DHCP server is responding on that VLAN
```

```ios
! Check associated clients on autonomous AP
AP# show dot11 associations
! Shows all connected clients and their statistics

! Check client details
AP# show dot11 associations all-client

! On WLC - check client association
(WLC)# show client summary
(WLC)# show client detail [MAC-ADDRESS]
```

---

## 4. Poor Wireless Performance / Slow Speeds

### Signal Strength Issues
```ios
! Check client signal strength on autonomous AP
AP# show dot11 associations
! Look for: Signal Strength and Signal Quality values
! Good: RSSI above -70 dBm, SNR above 20 dB

! Check radio statistics
AP# show interfaces Dot11Radio0
! Look for: input errors, output errors, retransmissions
```

### Channel Interference
```ios
! Check current channel configuration
AP# show controllers Dot11Radio0
! Look for: Current operating frequency

! Check for neighboring APs on same channel (autonomous)
AP# show dot11 scan results

! Change channel manually
AP(config)# interface Dot11Radio0
AP(config-if)# channel 6
```

**Non-overlapping 2.4 GHz channels:** 1, 6, 11
**Recommended 5 GHz channels:** 36, 40, 44, 48, 149, 153, 157, 161

### Common Performance Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Slow speeds near AP | 2.4 GHz congestion | Force clients to 5 GHz band |
| Slow speeds far from AP | Weak signal | Increase transmit power or add AP |
| Intermittent drops | Channel interference | Change to less congested channel |
| All clients slow | AP overloaded | Add another AP, enable load balancing |
| Specific client slow | Client driver issue | Update wireless driver on client |

---

## 5. Wireless Clients Dropping Connection

```ios
! Check deauthentication events on autonomous AP
AP# debug dot11 dot11radio0 trace print events

! Check for excessive retransmissions
AP# show interfaces Dot11Radio0 | include retry|error|retransmit

! Check client associations
AP# show dot11 associations

! On WLC - check for client roaming issues
(WLC)# show client roam-history [MAC-ADDRESS]
```

### Common Disconnect Causes

| Cause | Symptoms | Fix |
|-------|---------|-----|
| Weak signal | Drops only at range | Increase AP power or add AP |
| Channel interference | Random drops near AP | Change channel |
| IP address conflict | Drops on reconnect | Check DHCP leases |
| DHCP exhaustion | Can't reconnect | Expand DHCP pool |
| Idle timeout | Drops after inactivity | Adjust idle timeout policy |
| Roaming issues | Drops when moving | Configure fast roaming/802.11r |

---

## 6. Switch Port Configuration Issues

The AP's switch port must be correctly configured or VLAN traffic won't work.

```ios
! Check the AP's switch port configuration
SW-CORE-01# show interfaces FastEthernet 0/12 switchport
! Look for: mode (access or trunk) and VLAN assignment

! For single SSID - access port
SW-CORE-01(config)# interface FastEthernet 0/12
SW-CORE-01(config-if)# switchport mode access
SW-CORE-01(config-if)# switchport access vlan 10
SW-CORE-01(config-if)# spanning-tree portfast
SW-CORE-01(config-if)# no shutdown

! For multiple SSIDs on different VLANs - trunk port
SW-CORE-01(config)# interface FastEthernet 0/12
SW-CORE-01(config-if)# switchport mode trunk
SW-CORE-01(config-if)# switchport trunk native vlan 10
SW-CORE-01(config-if)# switchport trunk allowed vlan 10,40,99
SW-CORE-01(config-if)# spanning-tree portfast trunk
SW-CORE-01(config-if)# no shutdown
```

---

## 7. Autonomous AP - Common Diagnostic Commands

```ios
! AP software version
AP# show version

! Radio interface status
AP# show interfaces Dot11Radio0
AP# show interfaces Dot11Radio1

! Associated clients
AP# show dot11 associations

! SSID configuration
AP# show dot11 bssid

! IP configuration
AP# show ip interface brief

! CDP neighbors (connected switch)
AP# show cdp neighbors detail

! Running config
AP# show running-config

! Ping the gateway
AP# ping 192.168.1.1

! Check dot11 counters
AP# show dot11 statistics
```

---

## 8. Factory Reset

Use when the AP is completely misconfigured or the password is unknown.

### Autonomous AP Factory Reset
```
Method 1 - MODE button:
1. Unplug the AP ethernet cable
2. Hold the MODE button on the AP
3. Reconnect the ethernet cable while holding MODE
4. Hold for 20-30 seconds until the LED turns red
5. Release - AP will reset to factory defaults

Method 2 - Console:
AP# write erase
AP# reload
```

### Lightweight AP Factory Reset
```
Hold the MODE button for 20-30 seconds while powered
AP will reset and rejoin the WLC with default settings
```

---

## Common Issues & Quick Fix Reference

| Problem | First Check | Fix |
|---------|------------|-----|
| AP not powering on | `show power inline` on switch | Check PoE budget, try different port |
| Lightweight AP not joining WLC | `show ap join stats summary` | Configure DHCP Option 43 |
| Clients can't see SSID | `show dot11 bssid` | Check radio is up, SSID broadcast enabled |
| Wrong password rejected | Verify PSK in config | Check WPA version matches |
| Poor performance | `show dot11 associations` | Check RSSI, change channel |
| Clients dropping | Check retransmission counters | Change channel, check interference |
| No IP after connecting | Check DHCP on VLAN | Verify switch port VLAN and DHCP scope |
| AP LED solid red | Boot failure | Factory reset, check IOS image |

---

## Notes
- RSSI above -70 dBm is good - below -80 dBm will cause performance issues
- 5 GHz is almost always preferred over 2.4 GHz - push dual-band clients to 5 GHz
- Non-overlapping 2.4 GHz channels are 1, 6, and 11 - using any other causes overlap
- PoE+ (802.3at) is required for newer high-power APs - standard PoE (802.3af) may not be enough
- CAPWAP uses UDP 5246 and 5247 - if a firewall sits between AP and WLC both ports must be open
- Always check the switch port configuration first - most AP issues are actually switch port misconfigurations

---

## Related Documents
- [Cisco Access Point Configuration](cisco-access-point-configuration.md)
- [Cisco Switch Troubleshooting](cisco-switch-troubleshooting.md)
- [VLAN Configuration](vlan-configuration-cisco-switch.md)
