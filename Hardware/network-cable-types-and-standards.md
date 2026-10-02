# Network Cable Types and Standards
> A reference guide to ethernet cable categories, fiber optic types, connectors, and TIA-568 wiring standards.

**Category:** Hardware  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **TIA** | Telecommunications Industry Association | The organization that defines cabling standards |
| **TIA-568** | TIA-568 | The standard for commercial building telecommunications cabling |
| **Cat** | Category | The classification rating for ethernet cable performance |
| **UTP** | Unshielded Twisted Pair | Standard ethernet cable without shielding |
| **STP** | Shielded Twisted Pair | Ethernet cable with shielding to reduce interference |
| **FTP** | Foiled Twisted Pair | Cable with an overall foil shield |
| **S/FTP** | Screened Foiled Twisted Pair | Cable with both overall and individual pair shielding |
| **EMI** | Electromagnetic Interference | Electrical noise that can corrupt network signals |
| **Crosstalk** | Crosstalk | Signal interference between adjacent wire pairs |
| **Attenuation** | Signal Attenuation | Loss of signal strength over distance |
| **Bandwidth** | Bandwidth | The frequency range a cable can support |
| **RJ45** | Registered Jack 45 | The standard 8-pin connector for ethernet cables |
| **RJ11** | Registered Jack 11 | The smaller 4-pin connector used for telephone cables |
| **MDI** | Medium Dependent Interface | A straight-through port (on switches/APs) |
| **MDIX** | Medium Dependent Interface Crossover | A crossover port (on older devices - now mostly auto-sensing) |
| **568A** | T568A | One of two standard wiring patterns for RJ45 connectors |
| **568B** | T568B | The more common wiring pattern for RJ45 connectors in North America |
| **PoE** | Power over Ethernet | Delivering power through ethernet cable |
| **SMF** | Single-Mode Fiber | Fiber optic cable for long distances |
| **MMF** | Multi-Mode Fiber | Fiber optic cable for shorter distances |
| **SFP** | Small Form-factor Pluggable | A transceiver module for fiber or copper connections on switches |
| **LC** | Lucent Connector | A small fiber optic connector - most common in enterprise |
| **SC** | Subscriber Connector | A larger fiber optic connector - older enterprise environments |
| **ST** | Straight Tip | A bayonet-style fiber connector - mostly legacy |

---

## Overview
Choosing the right cable is critical for network performance and reliability. Using the wrong category, exceeding distance limits, or poor termination can cause intermittent failures that are extremely difficult to diagnose.

**Key principle:** Always install a cable category higher than your current needs - replacing cable inside walls is expensive.

---

## 1. Ethernet Cable Categories

### Category Comparison Table

| Category | Max Speed | Max Bandwidth | Max Distance | Use Case |
|----------|-----------|---------------|--------------|---------|
| **Cat 3** | 10 Mbps | 16 MHz | 100m | Legacy telephone, 10BASE-T (obsolete) |
| **Cat 5** | 100 Mbps | 100 MHz | 100m | Legacy - do not install (obsolete) |
| **Cat 5e** | 1 Gbps | 100 MHz | 100m | Minimum acceptable for new installs |
| **Cat 6** | 1 Gbps (10G up to 55m) | 250 MHz | 100m | Standard for new installations |
| **Cat 6A** | 10 Gbps | 500 MHz | 100m | Recommended for new builds, PoE++ |
| **Cat 7** | 10 Gbps | 600 MHz | 100m | Data centers, shielded |
| **Cat 8** | 25/40 Gbps | 2000 MHz | 30m | Data center top-of-rack only |

### Key Differences Explained

**Cat 5e vs Cat 6:**
- Cat 6 has tighter twists and a plastic separator (spline) between pairs
- Cat 6 handles 10 Gbps up to 55 meters - Cat 5e cannot do 10 Gbps
- Cat 6 has better crosstalk resistance
- Cat 6 is the minimum recommended for any new installation today

**Cat 6 vs Cat 6A:**
- Cat 6A is larger in diameter - requires larger conduit
- Cat 6A supports full 10 Gbps at 100 meters - Cat 6 only supports it to 55m
- Cat 6A is required for PoE++ (Type 4, up to 100W)
- Cat 6A is recommended for any installation that may need 10G in the future

---

## 2. TIA-568 Wiring Standards

TIA-568 defines how the 8 wires inside an ethernet cable are arranged in the RJ45 connector. There are two patterns - **T568A** and **T568B**.

### T568B (Most Common in North America)

```
Pin 1 - White/Orange  ──┐
Pin 2 - Orange        ──┘ Pair 2
Pin 3 - White/Green   ──┐
Pin 4 - Blue          ──┐ Pair 1
Pin 5 - White/Blue    ──┘
Pin 6 - Green         ──┘ Pair 3
Pin 7 - White/Brown   ──┐
Pin 8 - Brown         ──┘ Pair 4
```

### T568A

```
Pin 1 - White/Green   ──┐
Pin 2 - Green         ──┘ Pair 3
Pin 3 - White/Orange  ──┐
Pin 4 - Blue          ──┐ Pair 1
Pin 5 - White/Blue    ──┘
Pin 6 - Orange        ──┘ Pair 2
Pin 7 - White/Brown   ──┐
Pin 8 - Brown         ──┘ Pair 4
```

### Straight-Through vs Crossover

| Cable Type | End 1 | End 2 | Use |
|-----------|-------|-------|-----|
| **Straight-Through** | 568B | 568B | PC to switch, switch to router |
| **Crossover** | 568A | 568B | Switch to switch (old), PC to PC (old) |
| **Rollover/Console** | 568B | Reversed | Console cable for Cisco devices |

> **Note:** Modern network equipment uses **Auto-MDIX** - it automatically detects and corrects straight-through vs crossover. Crossover cables are rarely needed today.

### The Memory Trick for 568B
```
White/Orange, Orange, White/Green, Blue, White/Blue, Green, White/Brown, Brown
1            2       3            4     5            6      7            8
```

---

## 3. Cable Testing and Verification

### What a Cable Tester Checks
- **Continuity** - all 8 wires connected end to end
- **Wire map** - correct pin-to-pin connections (no crossed or reversed pairs)
- **Short** - wires touching each other
- **Split pair** - wires from different pairs terminated together (works but has terrible crosstalk)

### Common Wiring Faults

| Fault | Meaning | Cause |
|-------|---------|-------|
| **Open** | A wire is not connected | Broken wire, not punched down fully |
| **Short** | Two wires touching | Damaged cable, bad termination |
| **Reversed pair** | Pair wired backwards (e.g. pins 1&2 swapped) | Termination error |
| **Crossed pair** | Pair wired to wrong position | Termination error |
| **Split pair** | Wires from different pairs at same position | Using wrong color at wrong pin |

### PowerShell - Test Network Connectivity
```powershell
# Test basic connectivity
Test-NetConnection -ComputerName 192.168.1.1

# Test with detailed info
Test-NetConnection -ComputerName 192.168.1.1 -InformationLevel Detailed

# Check link speed (shows negotiated speed - can indicate cable issue)
Get-NetAdapter | Select-Object Name, LinkSpeed, Status

# Check for errors on adapter
Get-NetAdapterStatistics | Select-Object Name, ReceivedPackets, ReceivedErrors, SentPackets, OutboundErrors
```

---

## 4. PoE Power Requirements by Cable Category

| PoE Standard | Power | Pairs Used | Minimum Cable |
|-------------|-------|-----------|--------------|
| **802.3af** (PoE) | 15.4W | 2 pairs | Cat 5e |
| **802.3at** (PoE+) | 30W | 2 pairs | Cat 5e |
| **802.3bt Type 3** (PoE++) | 60W | 4 pairs | Cat 6A recommended |
| **802.3bt Type 4** (PoE++) | 100W | 4 pairs | Cat 6A required |

> Higher PoE standards generate more heat in the cable. Cat 6A is mandatory for 90W+ PoE - Cat 5e or Cat 6 will overheat in bundles.

---

## 5. Fiber Optic Cable Types

### Single-Mode Fiber (SMF)
- **Core diameter:** 9 microns
- **Color jacket:** Yellow
- **Distance:** Up to 40km+ depending on transceiver
- **Use:** WAN links, building-to-building, long runs
- **Cost:** More expensive transceivers (laser light source)

### Multi-Mode Fiber (MMF)

| Type | Core | Bandwidth | Max Distance at 10G | Jacket Color |
|------|------|-----------|--------------------:|-------------|
| **OM1** | 62.5μm | 200 MHz·km | 33m | Orange |
| **OM2** | 50μm | 500 MHz·km | 82m | Orange |
| **OM3** | 50μm | 2000 MHz·km | 300m | Aqua |
| **OM4** | 50μm | 4700 MHz·km | 400m | Aqua/Violet |
| **OM5** | 50μm | 28000 MHz·km | 400m (100G) | Lime Green |

**OM3 and OM4 are the current standard for new fiber installations.**

### Fiber Optic Connectors

| Connector | Description | Common Use |
|-----------|-------------|-----------|
| **LC** | Small form factor, push-pull latch | Enterprise switches, SFPs - most common |
| **SC** | Square body, push-pull | Older enterprise, patch panels |
| **ST** | Round, bayonet twist-lock | Legacy installations |
| **MPO/MTP** | Multi-fiber connector (12 or 24 fibers) | High-density data centers |
| **FC** | Screw-on, very secure | Telecom, testing equipment |

---

## 6. SFP Modules

SFP (Small Form-factor Pluggable) modules connect switches to fiber or long-distance copper runs.

| SFP Type | Speed | Distance | Media |
|---------|-------|----------|-------|
| **1000BASE-SX** | 1 Gbps | 550m | MMF (OM2+) |
| **1000BASE-LX** | 1 Gbps | 10km | SMF |
| **10GBASE-SR** | 10 Gbps | 400m | MMF (OM4) |
| **10GBASE-LR** | 10 Gbps | 10km | SMF |
| **1000BASE-T** | 1 Gbps | 100m | Cat 5e copper |
| **10GBASE-T** | 10 Gbps | 100m | Cat 6A copper |
| **DAC** | 1-100 Gbps | 1-7m | Direct attach copper |

---

## 7. Maximum Distance Rules

### Ethernet (Copper)
```
Maximum segment length: 100 meters (328 feet)
This includes:
- Horizontal cable (wall to patch panel): max 90 meters
- Patch cables (patch panel to switch + wall plate to device): max 10 meters combined

Never exceed 90 meters for permanent horizontal runs.
```

### Fiber Optic
```
OM3 at 10 Gbps: 300 meters
OM4 at 10 Gbps: 400 meters
SMF at 10 Gbps: 10,000 meters (10km)
SMF at 1 Gbps: up to 100km with appropriate transceiver
```

---

## 8. Cable Installation Best Practices

### Installation Do's
```
✅ Use Cat 6A for all new installations
✅ Maintain minimum bend radius (4x cable diameter for Cat 6A)
✅ Support cable every 1.5 meters in cable trays
✅ Keep ethernet away from power cables (minimum 30cm separation)
✅ Label both ends of every cable
✅ Test every cable after installation
✅ Document cable runs in a cable management system (NetBox)
✅ Leave service loops at both ends for future termination
✅ Use cable management rings in patch panels and racks
```

### Installation Don'ts
```
❌ Never exceed 100m total run length
❌ Never staple cables - use proper cable clips
❌ Never kink or sharply bend cables
❌ Never run ethernet parallel to high-voltage electrical for long distances
❌ Never untwist more than 13mm (0.5 inch) when terminating
❌ Never mix 568A and 568B on the same cable
❌ Never use Cat 5 for new installations
❌ Never use electrical tape to splice a damaged cable - replace it
```

---

## 9. Fiber Installation Best Practices

```
✅ Keep fiber bend radius above minimum (typically 10x cable diameter)
✅ Use appropriate strain relief at all termination points
✅ Clean fiber connectors before mating - use proper fiber cleaning tools
✅ Always cap unused fiber ports with dust caps
✅ Label fiber runs clearly - indicate SMF vs MMF and connector types
✅ Test with an OTDR after installation for long runs
✅ Use proper fiber enclosures in patch panels

❌ Never exceed minimum bend radius - fiber will crack or increase loss
❌ Never look directly into a fiber - laser light can cause eye damage
❌ Never touch the fiber end-face - oils from fingers degrade signal
❌ Never run fiber and copper in the same conduit if possible
```

---

## 10. Troubleshooting Cable Issues

### Symptoms of Cable Problems

| Symptom | Likely Cause |
|---------|-------------|
| Link doesn't come up | Open circuit, wrong connector, faulty cable |
| Link up but very slow | Split pair, excessive untwisting at termination |
| Intermittent connection | Damaged cable, loose termination, kink |
| Works at short distance but not long | Exceeds 100m limit, poor quality cable |
| High error rate | Crosstalk, interference, split pair |
| PoE device not powering | Cable doesn't support 4-pair PoE, damaged pairs |

### Cable Testing Tools

| Tool | What It Does |
|------|-------------|
| **Cable tester** (basic) | Verifies continuity and wire map |
| **Cable certifier** (advanced) | Verifies category compliance - tests attenuation, crosstalk, return loss |
| **TDR** (Time Domain Reflectometer) | Locates faults and measures cable length |
| **OTDR** (Optical TDR) | Tests fiber optic runs for faults and loss |
| **Visual fault locator** | Shines visible light through fiber to find breaks |
| **Fluke Networks DSX** | The industry standard cable certifier |

---

## Quick Reference

### TIA-568B Pin Order (Most Common)
```
1 = White/Orange
2 = Orange
3 = White/Green
4 = Blue
5 = White/Blue
6 = Green
7 = White/Brown
8 = Brown
```

### Cable Category Quick Pick
```
Desktop/phone runs        -> Cat 6
High-density areas        -> Cat 6A
PoE++ devices (60W+)      -> Cat 6A
Short fiber (building)    -> OM3 or OM4 MMF
Long fiber (campus/WAN)   -> SMF
Data center spine         -> OM4 or SMF
```

### Maximum Distances
```
Cat 5e/6/6A copper   -> 100m total (90m permanent run)
OM3 at 10G           -> 300m
OM4 at 10G           -> 400m
SMF at 10G           -> 10km
```

---

## Notes
- TIA-568B is the standard in North America - use it consistently throughout your installation
- Never mix 568A and 568B on the same cable - this creates a crossover cable
- Exceeding the 100m limit is one of the most common and hardest to diagnose network problems
- Always test cables after installation - visual inspection is not sufficient
- OM3 and OM4 look the same (aqua) - label them clearly
- Fiber connectors must be clean - a dirty connector can cause more signal loss than a long cable run

---

## Related Documents
- [Cisco Switch Configuration](cisco-switch-configuration.md)
- [Cisco Switch Troubleshooting](cisco-switch-troubleshooting.md)
- [Cisco Access Point Configuration](cisco-access-point-configuration.md)
