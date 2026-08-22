# Cisco Access Point Configuration
> A guide to configuring and managing Cisco Aironet and Catalyst wireless access points.

**Category:** Networking  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **AP** | Access Point | A wireless device that connects Wi-Fi clients to a wired network |
| **SSID** | Service Set Identifier | The name of a wireless network |
| **BSSID** | Basic Service Set Identifier | The MAC address of a specific AP radio |
| **WLC** | Wireless LAN Controller | A centralized controller managing multiple APs |
| **CAPWAP** | Control and Provisioning of Wireless Access Points | The protocol used between APs and WLC |
| **Lightweight AP** | Lightweight AP | An AP that is managed by a WLC — cannot operate standalone |
| **Autonomous AP** | Autonomous AP | An AP that manages itself — no WLC required |
| **2.4 GHz** | 2.4 GHz Band | Longer range, slower speed, more interference |
| **5 GHz** | 5 GHz Band | Shorter range, faster speed, less interference |
| **6 GHz** | 6 GHz Band | Newest band — fastest speed, shortest range (Wi-Fi 6E) |
| **802.11** | IEEE 802.11 | The family of wireless networking standards |
| **Wi-Fi 5** | 802.11ac | Up to 3.5 Gbps, 5 GHz only |
| **Wi-Fi 6** | 802.11ax | Up to 9.6 Gbps, 2.4 and 5 GHz |
| **WPA3** | Wi-Fi Protected Access 3 | Current wireless security standard |
| **WPA2** | Wi-Fi Protected Access 2 | Previous standard — still widely used |
| **PSK** | Pre-Shared Key | A password used for WPA2/WPA3 Personal authentication |
| **802.1X** | 802.1X | Enterprise authentication using a RADIUS server |
| **PoE** | Power over Ethernet | Delivers electrical power through the network cable — how most APs are powered |
| **Channel** | Wireless Channel | A specific frequency within a band used for communication |
| **RSSI** | Received Signal Strength Indicator | A measure of wireless signal strength |
| **Roaming** | Wireless Roaming | A device moving between APs while staying connected |

---

## Overview
Cisco Access Points provide wireless connectivity for devices on your network. They connect to switches via ethernet (usually powered by PoE) and broadcast one or more wireless SSIDs that devices connect to.

**Wireless Standards:**

| Standard | Max Speed | Frequency |
|---------|-----------|-----------|
| 802.11b | 11 Mbps | 2.4 GHz |
| 802.11g | 54 Mbps | 2.4 GHz |
| 802.11n (Wi-Fi 4) | 600 Mbps | 2.4 & 5 GHz |
| 802.11ac (Wi-Fi 5) | 3.5 Gbps | 5 GHz |
| 802.11ax (Wi-Fi 6) | 9.6 Gbps | 2.4, 5 & 6 GHz |

---

## Prerequisites
- Cisco Access Point (Aironet or Catalyst series)
- PoE switch or PoE injector to power the AP
- Console cable for initial setup (or DHCP for automatic provisioning)
- For Lightweight AP — a Wireless LAN Controller (WLC)

---

## Deployment Types

### Autonomous AP (Standalone)
Each AP is configured individually — suitable for small deployments (1-5 APs).

### Lightweight AP + WLC (Controller-Based)
APs are managed centrally by a WLC — suitable for medium to large deployments (5+ APs). The AP joins the WLC automatically via CAPWAP.

---

## 1. Physical Installation

```
1. Mount AP on ceiling or wall — away from interference sources
2. Run ethernet cable from AP to PoE switch port
3. Connect ethernet — AP powers on automatically via PoE
4. Wait 2-3 minutes for AP to boot
5. AP LED indicates status:
   - Blinking Green  → Normal operation
   - Solid Green     → Associated with WLC
   - Blinking Amber  → Booting or not associated
   - Red             → Error
```

---

## 2. Connecting to AP via Console

```
Settings: 9600 baud, 8N1, no flow control

! Default credentials (first boot)
Username: Cisco
Password: Cisco

! Or check the label on the AP
```

---

## 3. Configuring an Autonomous AP

### Basic Setup
```ios
! Enter enable mode
ap> enable
Password: Cisco
ap#

! Enter config mode
ap# configure terminal

! Set hostname
ap(config)# hostname AP-OFFICE-01

! Set enable secret
AP-OFFICE-01(config)# enable secret StrongPass123

! Set management IP (if not using DHCP)
AP-OFFICE-01(config)# interface BVI1
AP-OFFICE-01(config-if)# ip address 192.168.1.20 255.255.255.0
AP-OFFICE-01(config-if)# no shutdown
AP-OFFICE-01(config-if)# exit

! Set default gateway
AP-OFFICE-01(config)# ip default-gateway 192.168.1.1
```

### Configuring an SSID
```ios
! Create the SSID
AP-OFFICE-01(config)# dot11 ssid COMPANY-WIFI
AP-OFFICE-01(config-ssid)# authentication open
AP-OFFICE-01(config-ssid)# authentication key-management wpa version 2
AP-OFFICE-01(config-ssid)# wpa-psk ascii CompanyWiFiPass123
AP-OFFICE-01(config-ssid)# guest-mode   ! makes SSID visible (broadcast)
AP-OFFICE-01(config-ssid)# exit

! Apply SSID to the radio interface (2.4 GHz)
AP-OFFICE-01(config)# interface Dot11Radio0
AP-OFFICE-01(config-if)# ssid COMPANY-WIFI
AP-OFFICE-01(config-if)# no shutdown
AP-OFFICE-01(config-if)# exit

! Apply SSID to the radio interface (5 GHz)
AP-OFFICE-01(config)# interface Dot11Radio1
AP-OFFICE-01(config-if)# ssid COMPANY-WIFI
AP-OFFICE-01(config-if)# no shutdown
AP-OFFICE-01(config-if)# exit

! Save
AP-OFFICE-01# wr
```

### Configuring a Guest SSID on a Separate VLAN
```ios
! Create guest SSID
AP-OFFICE-01(config)# dot11 ssid GUEST-WIFI
AP-OFFICE-01(config-ssid)# vlan 40
AP-OFFICE-01(config-ssid)# authentication open
AP-OFFICE-01(config-ssid)# authentication key-management wpa version 2
AP-OFFICE-01(config-ssid)# wpa-psk ascii GuestPass456
AP-OFFICE-01(config-ssid)# guest-mode
AP-OFFICE-01(config-ssid)# exit

! Create subinterface for guest VLAN on radio
AP-OFFICE-01(config)# interface Dot11Radio0.40
AP-OFFICE-01(config-subif)# encapsulation dot1Q 40
AP-OFFICE-01(config-subif)# bridge-group 40
AP-OFFICE-01(config-subif)# exit

! Create subinterface on ethernet for guest VLAN
AP-OFFICE-01(config)# interface FastEthernet0.40
AP-OFFICE-01(config-subif)# encapsulation dot1Q 40
AP-OFFICE-01(config-subif)# bridge-group 40
AP-OFFICE-01(config-subif)# exit
```

---

## 4. Configuring Channel and Power

```ios
! Set channel on 2.4 GHz radio (use 1, 6, or 11 for non-overlapping)
AP-OFFICE-01(config)# interface Dot11Radio0
AP-OFFICE-01(config-if)# channel 6
AP-OFFICE-01(config-if)# exit

! Set channel on 5 GHz radio
AP-OFFICE-01(config)# interface Dot11Radio1
AP-OFFICE-01(config-if)# channel 36
AP-OFFICE-01(config-if)# exit

! Set transmit power (1=max, 8=min on most APs)
AP-OFFICE-01(config)# interface Dot11Radio0
AP-OFFICE-01(config-if)# power local 50
AP-OFFICE-01(config-if)# exit
```

**Non-overlapping 2.4 GHz channels:**
```
Channel 1  — 2.412 GHz
Channel 6  — 2.437 GHz
Channel 11 — 2.462 GHz
```

**Common 5 GHz channels (UNII-1 and UNII-3):**
```
36, 40, 44, 48, 149, 153, 157, 161, 165
```

---

## 5. Enabling SSH on the AP

```ios
AP-OFFICE-01(config)# ip domain-name company.com
AP-OFFICE-01(config)# username admin privilege 15 secret AdminPass123
AP-OFFICE-01(config)# crypto key generate rsa modulus 2048
AP-OFFICE-01(config)# ip ssh version 2
AP-OFFICE-01(config)# line vty 0 4
AP-OFFICE-01(config-line)# transport input ssh
AP-OFFICE-01(config-line)# login local
AP-OFFICE-01(config-line)# exit
AP-OFFICE-01# wr
```

---

## 6. Lightweight AP — Joining a WLC

Lightweight APs automatically discover and join a WLC. The process is mostly automatic.

### Discovery Methods (in order)
1. **DHCP Option 43** — DHCP server sends WLC IP to the AP
2. **DNS** — AP resolves `CISCO-CAPWAP-CONTROLLER.localdomain`
3. **Broadcast** — AP broadcasts on local subnet
4. **Primed** — WLC IP was manually configured

### Configuring DHCP Option 43 on Windows DHCP Server
```powershell
# Add Option 43 to DHCP scope with WLC IP
# WLC IP: 192.168.1.100 in hex = c0a80164
# Format: f1:04:c0:a8:01:64

Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 `
  -OptionId 43 `
  -Value "f104c0a80164"
```

### Verifying AP joined WLC
```ios
! On the WLC CLI
(WLC)# show ap summary
(WLC)# show ap join stats summary all
(WLC)# show ap config general AP-NAME
```

---

## 7. Verifying AP Operation

```ios
! Check radio status
AP-OFFICE-01# show interfaces Dot11Radio0
AP-OFFICE-01# show interfaces Dot11Radio1

! Check associated clients
AP-OFFICE-01# show dot11 associations

! Check SSID configuration
AP-OFFICE-01# show dot11 bssid

! Check running config
AP-OFFICE-01# show running-config

! Check IP connectivity
AP-OFFICE-01# ping 192.168.1.1

! Check CDP neighbors (connected switch)
AP-OFFICE-01# show cdp neighbors
```

---

## 8. Switch Port Configuration for APs

The switch port connecting the AP must be configured correctly for VLANs to work.

```ios
! If using a single SSID (access port)
SW-CORE-01(config)# interface FastEthernet 0/12
SW-CORE-01(config-if)# description AP-OFFICE-01
SW-CORE-01(config-if)# switchport mode access
SW-CORE-01(config-if)# switchport access vlan 10
SW-CORE-01(config-if)# spanning-tree portfast
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit

! If using multiple SSIDs on different VLANs (trunk port)
SW-CORE-01(config)# interface FastEthernet 0/12
SW-CORE-01(config-if)# description AP-OFFICE-01
SW-CORE-01(config-if)# switchport mode trunk
SW-CORE-01(config-if)# switchport trunk native vlan 10
SW-CORE-01(config-if)# switchport trunk allowed vlan 10,40,99
SW-CORE-01(config-if)# spanning-tree portfast trunk
SW-CORE-01(config-if)# no shutdown
SW-CORE-01(config-if)# exit
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| AP not powering on | Switch port not PoE or PoE budget exceeded | Verify switch port has PoE, check PoE budget with `show power inline` |
| AP not broadcasting SSID | Radio disabled or SSID not applied to radio | Check `show interfaces Dot11Radio0`, apply SSID to radio |
| Clients can't connect | Wrong password or security mismatch | Verify WPA2/3 settings and PSK |
| Poor signal in some areas | AP placement or power too low | Reposition AP or increase transmit power |
| Lightweight AP not joining WLC | Wrong WLC IP or CAPWAP blocked | Check DHCP option 43 or DNS, check firewall allows UDP 5246/5247 |
| Channel interference | Multiple APs on same channel | Use non-overlapping channels (1, 6, 11) for 2.4 GHz |
| VLAN traffic not passing through AP | Switch port misconfigured | Check switch port is trunk with correct VLANs allowed |

---

## Quick Reference

```ios
! View radio status
show interfaces Dot11Radio0

! View associated clients
show dot11 associations

! View SSIDs
show dot11 bssid

! Set channel
interface Dot11Radio0
channel 6

! Create SSID
dot11 ssid WIFI-NAME
authentication open
authentication key-management wpa version 2
wpa-psk ascii PASSWORD
guest-mode

! Apply SSID to radio
interface Dot11Radio0
ssid WIFI-NAME

! Save config
wr
```

---

## Notes
- Always use PoE+ (802.3at) for newer APs — standard PoE (802.3af) may not provide enough power
- Use non-overlapping channels on 2.4 GHz — 1, 6, and 11
- 5 GHz is preferred for performance — more channels, less interference, faster speeds
- WPA3 is the current standard — use WPA2 only if WPA3 is not supported by all clients
- Place APs every 20-30 meters for good coverage in typical office environments
- CAPWAP uses UDP ports 5246 (control) and 5247 (data) — these must be open between AP and WLC
- `spanning-tree portfast` on AP switch ports speeds up the port coming up

---

## Related Documents
- [Cisco Switch Configuration](cisco-switch-configuration.md)
- [VLAN Configuration](vlan-configuration-cisco-switch.md)
- [Cisco Access Point Troubleshooting](../Hardware/cisco-access-troubleshooting.md)
