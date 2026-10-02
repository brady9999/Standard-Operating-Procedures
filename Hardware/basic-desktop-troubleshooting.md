# Basic Desktop Troubleshooting
> A guide to diagnosing and resolving common desktop computer hardware and software issues.

**Category:** Hardware  
**Last Updated:** 2026-09-29
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **POST** | Power-On Self-Test | The diagnostic the computer runs when first powered on |
| **PSU** | Power Supply Unit | Converts AC power to DC power for all internal components |
| **GPU** | Graphics Processing Unit | The dedicated graphics card |
| **CPU** | Central Processing Unit | The main processor |
| **Beep Code** | POST Beep Code | Audible codes from the motherboard speaker indicating hardware failures |
| **CMOS** | Complementary Metal-Oxide Semiconductor | Stores BIOS settings - reset by removing the CMOS battery |
| **ATX** | Advanced Technology eXtended | The standard form factor for desktop motherboards and power supplies |
| **PCIe** | Peripheral Component Interconnect Express | The slot used for GPUs, NVMe SSDs, and expansion cards |
| **SATA** | Serial ATA | Interface for connecting HDDs, SSDs, and optical drives |
| **Front Panel Header** | Front Panel Connector | The small connectors from the case to the motherboard (power button, LEDs) |

---

## Overview

**Basic troubleshooting order:**
1. Check physical connections
2. Verify power
3. Isolate to hardware or software
4. Test components individually
5. Replace failed component

---

## Troubleshooting Methodology

```
1. IDENTIFY - What exactly is the problem?
2. CHECK PHYSICAL - Cables seated? Components secure? Damage visible?
3. SIMPLIFY - Remove non-essential components and test
4. SWAP - Replace suspected component with known-good
5. FIX - Apply solution
6. VERIFY - Confirm resolved
7. DOCUMENT - Record problem and fix
```

---

## 1. Desktop Won't Turn On

### Step by Step
1. Check the power cable is plugged into the PSU and the wall outlet
2. Check the PSU switch on the back is in the **ON (I)** position - not **(O)**
3. Try a different wall outlet or power strip
4. Check the power button connector is attached to the motherboard header
5. Press the power button - listen for any fans, drives, or beeps

### No Response At All
```
Possible causes:
- Dead PSU -> Test with PSU tester or known-good PSU
- Dead wall outlet -> Test with another device
- Front panel power button not connected -> Check header pins
- Blown fuse in power cable -> Replace cable
```

### Powers On But No Display
```
Possible causes:
- RAM not seated -> Reseat RAM sticks
- GPU not seated -> Reseat GPU
- Monitor not connected properly -> Check cable and input source
- CMOS issue -> Clear CMOS
```

### Beep Codes During POST

| Beeps | Common Meaning (BIOS dependent) |
|-------|--------------------------------|
| 1 short | POST successful - normal |
| 1 long, 2 short | Video card error (AMI BIOS) |
| 2 short | Memory error (Award BIOS) |
| 3 long | Memory error (AMI BIOS) |
| Continuous | RAM or power issue |
| No beep, no display | No RAM or dead CPU/motherboard |

---

## 2. Power Supply Diagnostics 

### Paper Clip Test (PSU Only - No Motherboard) (Last Resort)
1. Unplug PSU from everything
2. Find the 24-pin ATX connector
3. Short the green wire (PS_ON) to any black wire (Ground) with a paper clip
4. Plug PSU into wall and flip the switch
5. If fans spin - PSU has basic functionality
6. Use a multimeter to verify voltages:

| Rail | Expected Voltage | Tolerance |
|------|-----------------|-----------|
| +3.3V | 3.3V | ±5% |
| +5V | 5V | ±5% |
| +12V | 12V | ±5% |
| -12V | -12V | ±10% |
| +5VSB | 5V | ±5% |

```powershell
# Check PSU via Windows (limited info)
Get-WmiObject Win32_Battery  # For UPS/battery backup systems

# Monitor system voltages 
# Or check motherboard utility software
```

---

## 3. Display Issues

### No Display / Black Screen
1. Check monitor is powered on and input source is correct
2. Check video cable (HDMI, DisplayPort, DVI, VGA) is securely connected at both ends
3. Try a different cable
4. Try a different monitor
5. If using dedicated GPU - try the motherboard's integrated video output
6. Reseat the GPU in the PCIe slot

```powershell
# View display adapters
Get-WmiObject Win32_VideoController | Select-Object Name, Status, DriverVersion

# Check display connection
Get-WmiObject Win32_DesktopMonitor | Select-Object Name, ScreenHeight, ScreenWidth

# Update display drivers
pnputil /scan-devices
```

---

## 4. Performance Issues

### Overheating Desktop
```powershell
# Check CPU temperatures
Get-WmiObject MSAcpi_ThermalZoneTemperature -Namespace root/wmi |
  ForEach-Object { [math]::Round(($_.CurrentTemperature / 10) - 273.15, 1) }
```

### Desktop Is Running Slow
**Physical checks:**
1. Open the case - check for excessive dust on fans and heatsinks
2. Verify all case fans are spinning
3. Check CPU fan is spinning and properly attached to heatsink
4. Ensure airflow is logical - intake fans in front, exhaust fans in back/top
5. Clean all dust filters

**Logical checks:**
1. Open Task Manager 
2. Processes Tab - read what applications have the most RAM and CPU usage
3. Right click un-needed application, click end task
4. Click Performance and check if usage on RAM and CPU went down

### When Task Manager Is Blocked
```powershell
# Check what's consuming resources
Get-Process | Sort-Object CPU -Descending | Select-Object -First 15 Name, CPU, WorkingSet

# Check disk usage
Get-Counter "\PhysicalDisk(*)\% Disk Time" -SampleInterval 2 -MaxSamples 5

# Check available RAM
Get-WmiObject Win32_OperatingSystem |
  Select-Object @{n='TotalGB';e={[math]::Round($_.TotalVisibleMemorySize/1MB,2)}},
               @{n='FreeGB';e={[math]::Round($_.FreePhysicalMemory/1MB,2)}}

# Check disk space
Get-PSDrive -PSProvider FileSystem | Select-Object Name, @{n='UsedGB';e={[math]::Round($_.Used/1GB,2)}}, @{n='FreeGB';e={[math]::Round($_.Free/1GB,2)}}

# View startup programs
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command

# Disable startup programs via Task Manager
# Or via PowerShell (disable from registry)
```


---

## 5. Storage Issues

```powershell
# Check drive health
Get-PhysicalDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus

# Check for disk errors
chkdsk C: /scan

# Full disk check (requires restart)
chkdsk C: /f /r

# View disk info
Get-Disk | Select-Object Number, FriendlyName, Size, HealthStatus

# View volumes
Get-Volume | Select-Object DriveLetter, FileSystemLabel, Size, SizeRemaining, HealthStatus
```

---

## 6. RAM Issues

```powershell
# View installed RAM
Get-WmiObject Win32_PhysicalMemory |
  Select-Object BankLabel, Capacity, Speed, Manufacturer, PartNumber

# Total RAM
(Get-WmiObject Win32_ComputerSystem).TotalPhysicalMemory / 1GB

# Run Windows Memory Diagnostic
MdSched.exe
```

**Physical RAM troubleshooting:**
1. Power off and unplug
2. Remove all RAM sticks
3. Install one stick in slot 1 (check motherboard manual)
4. Power on - if it boots, RAM and slot are good
5. Add second stick - if it crashes, second stick or second slot is faulty
6. Rotate sticks and slots to isolate the bad component

---

## 7. Network Issues

```powershell
# View network adapters
Get-NetAdapter | Select-Object Name, Status, LinkSpeed

# Test internet connectivity
Test-NetConnection -ComputerName "8.8.8.8"
Test-NetConnection -ComputerName "google.com" -Port 443

# View IP configuration
ipconfig /all

# Reset network
netsh int ip reset
netsh winsock reset
ipconfig /flushdns

# Renew IP
ipconfig /release
ipconfig /renew

# Ping with packet loss check
Test-Connection 8.8.8.8 -Count 20 | Measure-Object ResponseTime -Average -Maximum
```

---

## 8. Audio Issues

```powershell
# Check audio devices
Get-WmiObject Win32_SoundDevice | Select-Object Name, Status

# Restart audio service
Restart-Service AudioSrv
Restart-Service AudioEndpointBuilder

# Check audio policy service
Get-Service | Where-Object {$_.DisplayName -like "*audio*"}
```

**Physical checks:**
- Check if any app is muted in volume mixer
- Verify speakers/headphones are plugged into the correct port (green = audio out)
- Check if speaker power is on (powered desktop speakers)
- Try headphones to isolate speaker vs system audio

---

## 9. USB Issues

```powershell
# View USB devices
Get-PnpDevice | Where-Object {$_.Class -eq "USB"} | Select-Object FriendlyName, Status

# View USB devices with issues
Get-PnpDevice | Where-Object {$_.Class -eq "USB" -and $_.Status -ne "OK"} | Select-Object FriendlyName, Status, Problem

# Reset USB controller (via Device Manager)
# Uninstall USB Root Hubs -> Scan for hardware changes
```

**Physical checks:**
- Try a different USB port
- Try a different USB cable
- Test the device on another computer
- Check if USB ports are enabled in BIOS

---

## 10. System Repair Commands

```powershell
# System File Checker - repairs corrupted Windows system files
sfc /scannow

# DISM - repairs Windows image
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
DISM /Online /Cleanup-Image /RestoreHealth

# Check disk
chkdsk C: /f /r

# Reset network stack
netsh int ip reset
netsh winsock reset

# Rebuild BCD (from recovery environment)
bootrec /fixmbr
bootrec /fixboot
bootrec /rebuildbcd

# System restore (GUI)
rstrui.exe

# Generate system info
msinfo32 /report "C:\systeminfo.txt"
```

---

## Common Issues & Quick Fix Reference

| Problem | First Check | Command/Action |
|---------|------------|----------------|
| Won't turn on | PSU switch, cables | Check power button header |
| No display | Monitor input, GPU seated | Try integrated GPU |
| Slow | Task Manager | `Get-Process \| Sort CPU` |
| Network down | Cable, adapter status | `ipconfig /release /renew` |
| Audio gone | Service running | `Restart-Service AudioSrv` |
| BSoD | Error code, Event Viewer | `sfc /scannow` |
| Disk errors | SMART status | `chkdsk C: /f /r` |
| USB not working | Different port, cable | Device Manager - rescan |

---

## Notes
- Always power off and unplug before touching internal components
- Ground yourself before touching components - ESD can kill hardware silently
- Desktop troubleshooting is much easier than laptop - components are accessible and swappable
- A POST code display card is invaluable - shows exactly where POST is failing
- When in doubt - remove everything and add components back one at a time

---
