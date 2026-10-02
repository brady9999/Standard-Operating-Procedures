# Enhanced Desktop Troubleshooting
> Advanced hardware diagnostics, component testing, and system recovery for desktop computers.

**Category:** Hardware  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **VRM** | Voltage Regulator Module | Regulates power delivered to the CPU on the motherboard |
| **POST Card** | POST Diagnostic Card | A card that displays POST codes to help identify where boot fails |
| **Multimeter** | Multimeter | A device for measuring voltage, current, and resistance |
| **Capacitor** | Capacitor | An electronic component that stores charge - can fail and bulge |
| **Thermal Compound** | Thermal Paste | Heat-conducting material between CPU and heatsink |
| **Load Line Calibration** | LLC | A BIOS setting affecting CPU voltage stability under load |
| **XMP/EXPO** | Extreme Memory Profile / Extended Profiles for Overclocking | BIOS settings to run RAM at rated speeds |
| **IOMMU** | Input–Output Memory Management Unit | Hardware for virtualization and device isolation |
| **RAID** | Redundant Array of Independent Disks | Combining multiple drives for redundancy or performance |
| **RAID Controller** | RAID Controller | Hardware or software managing a RAID array |
| **Boot Order** | Boot Priority | The sequence of devices the BIOS tries to boot from |
| **Secure Boot** | Secure Boot | UEFI feature that prevents unsigned bootloaders from running |
| **TPM** | Trusted Platform Module | A security chip for storing keys and enabling features like BitLocker |

---

## Overview
Enhanced desktop troubleshooting involves deep hardware diagnosis - component-level testing, voltage measurement, POST code reading, and advanced Windows diagnostics. This level is used when basic troubleshooting has failed to identify or resolve the issue.

---

## 1. POST Code Diagnostics

A POST code display card plugs into a PCIe or PCI slot and shows a two-digit hex code indicating exactly where the boot process is failing.

### Common POST Codes (AMI BIOS)

| Code Range | Stage |
|-----------|-------|
| 00-0F | Pre-memory initialization |
| 10-1F | Pre-memory CPU init |
| 20-2F | Memory initialization |
| 30-3F | Post-memory CPU init |
| 40-4F | DXE initialization |
| 50-5F | DXE driver dispatch |
| 60-6F | PCI enumeration |
| 70-7F | DXE post dispatch |
| 90-9F | Boot Device Selection |
| A0-AF | Pre-OS boot |
| **00 / FF** | System hang - no code - usually CPU or motherboard |

### Common Problematic Codes

| Code | Meaning |
|------|---------|
| 00 | CPU not detected or dead |
| 0d | No CPU or bad power |
| 15 | Pre-memory initialization failing |
| 55 | Memory not detected |
| d6 | No console output device - GPU issue |
| 99 | Super I/O initialization - usually benign |

---

## 2. PSU Advanced Testing

### Multimeter Voltage Testing
With PSU running (paperclip test or connected to system):

```
24-pin ATX Connector:
Pin 1  (Orange)  = +3.3V  -> Should read 3.135V to 3.465V
Pin 4  (Red)     = +5V    -> Should read 4.75V to 5.25V
Pin 9  (Purple)  = +5VSB  -> Should read 4.75V to 5.25V
Pin 10 (Yellow)  = +12V   -> Should read 11.4V to 12.6V
Pin 11 (Orange)  = +3.3V  -> Should read 3.135V to 3.465V

8-pin CPU Connector:
All Yellow = +12V  -> Should read 11.4V to 12.6V
```

### PSU Load Testing
A PSU may test fine with no load but fail under load. Use a PSU load tester or connect to the system and run a stress test while monitoring voltages in HWiNFO.

Signs of a failing PSU under load:
- System randomly shuts off during gaming or rendering
- Voltages drop significantly under load
- Coil whine that changes with load
- Random restarts or BSoDs only during heavy use

---

## 3. GPU Diagnostics

```powershell
# View GPU info
Get-WmiObject Win32_VideoController |
  Select-Object Name, DriverVersion, AdapterRAM, VideoProcessor, Status

# Check GPU events
Get-WinEvent -LogName "System" | Where-Object {$_.Message -like "*display*" -or $_.Message -like "*GPU*"} |
  Select-Object TimeCreated, Message -First 20

# Check for TDR events (GPU timeout and recovery)
Get-WinEvent -FilterHashtable @{LogName='System'; Id=4101} |
  Select-Object TimeCreated, Message
```

### GPU Stress Testing
1. Run FurMark for GPU stress testing
2. Monitor GPU temperature - should stay below 90°C
3. Watch for:
   - Screen artifacts (visual glitches) - VRAM issue
   - Driver crash (TDR) - driver or GPU hardware issue
   - System crash - PSU or GPU hardware issue

### GPU Seating and Power
```
Physical checks:
1. Reseat GPU in PCIe slot - press until it clicks
2. Verify all PCIe power connectors are firmly attached (6-pin, 8-pin, 12-pin)
3. Check GPU is in the primary PCIe slot (usually the top x16 slot)
4. Try GPU in a different PCIe slot if available
5. Test GPU in another system
```

---

## 4. RAM Advanced Testing

```powershell
# Detailed RAM info
Get-WmiObject Win32_PhysicalMemory |
  Select-Object BankLabel, DeviceLocator, Capacity, Speed, Manufacturer, PartNumber, SerialNumber

# Check if XMP/EXPO is enabled (RAM running at rated speed)
Get-WmiObject Win32_PhysicalMemory | Select-Object Speed
# Compare to sticker speed on RAM - if lower, XMP not enabled in BIOS

# Memory performance test
winsat mem
```

### MemTest86 Interpretation
- **Pass 0:** Basic tests - should complete quickly
- **Passes 1-7:** Thorough tests - minimum 2 full passes recommended
- **Any error:** Even 1 error = RAM is faulty
- **Errors only in certain slots:** Could be bad slot on motherboard
- **Errors in all configs:** RAM sticks are faulty

### Dual Channel Testing
```
Proper dual channel slot placement (varies by board - check manual):
Most Intel boards: A2 + B2 (slots 2 and 4)
Most AMD boards: A2 + B2 (slots 2 and 4)

If system is unstable with 2 sticks but stable with 1:
-> Try different slot combinations
-> Check motherboard QVL (Qualified Vendor List) for RAM compatibility
```

---

## 5. Storage Advanced Diagnostics

### SMART Deep Dive
```powershell
# Get detailed SMART data (requires third-party tool like CrystalDiskInfo)
# Or basic health via PowerShell
Get-PhysicalDisk | Select-Object *

# StorageReliabilityCounter (Windows 10/11 Server)
Get-StorageReliabilityCounter | Select-Object *
```

### RAID Diagnostics
```powershell
# View storage spaces (Windows software RAID)
Get-StoragePool | Select-Object FriendlyName, HealthStatus, OperationalStatus

# View virtual disks in pools
Get-VirtualDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus, ResiliencySettingName

# View physical disks in pool
Get-StoragePool -FriendlyName "PoolName" | Get-PhysicalDisk |
  Select-Object FriendlyName, HealthStatus, Usage
```

### Recovering Data from Failing Drive
```powershell
# If drive is still readable - copy data immediately
robocopy C:\Users\Username\Documents D:\Backup\Documents /E /LOG:"C:\copy-log.txt" /R:3 /W:5

# Check and skip bad sectors during copy
robocopy C:\Source D:\Dest /E /R:0 /W:0
```

---

## 6. Motherboard Diagnostics

### CMOS Reset
Resets BIOS to factory defaults - fixes many boot and instability issues:

**Method 1 - BIOS menu:**
Settings -> Load Defaults -> Save and Exit

**Method 2 - CMOS jumper:**
1. Power off and unplug
2. Locate the CMOS jumper (usually labeled CLR_CMOS or JBAT)
3. Move jumper from pins 1-2 to pins 2-3 for 10 seconds
4. Return jumper to original position

**Method 3 - Remove CMOS battery:**
1. Power off and unplug
2. Locate the coin cell battery on the motherboard
3. Remove for 60 seconds
4. Reinsert

### Checking Capacitors
Visually inspect capacitors on the motherboard for:
- Bulging tops (should be flat)
- Leaking electrolyte (brown crusty residue around base)
- Burn marks

Blown capacitors = motherboard needs replacement or professional repair.

### Testing with Minimum Components
Strip the system down to:
- Motherboard
- CPU + CPU cooler
- 1 RAM stick
- PSU
- GPU (or use integrated)
- Monitor

If this boots, add components back one at a time to identify what causes the failure.

---

## 7. Advanced Windows Diagnostics

```powershell
# Reliability Monitor - shows timeline of crashes and errors
# Open via: Control Panel -> Security and Maintenance -> View reliability history
# Or run:
Start-Process "C:\Windows\system32\mmc.exe" "/a perfmon.msc /s"

# Performance Monitor - real-time performance data
perfmon.exe

# Resource Monitor - detailed CPU, RAM, disk, network usage
resmon.exe

# Generate complete system diagnostic report
# Note: Takes 60 seconds
Start-Process powershell -ArgumentList "-Command perfmon /report" -Wait

# View all system events in last hour
$hour = (Get-Date).AddHours(-1)
Get-WinEvent -FilterHashtable @{LogName='System','Application'; StartTime=$hour; Level=1,2} |
  Select-Object TimeCreated, LogName, ProviderName, Message

# Check for hardware errors
Get-WinEvent -LogName "System" | Where-Object {$_.ProviderName -like "*Hardware*" -or $_.ProviderName -like "*Disk*"} |
  Select-Object TimeCreated, ProviderName, Message -First 20
```

---

## 8. BIOS/UEFI Advanced Settings

### Settings That Cause Instability

| Setting | Issue if Wrong | Fix |
|---------|---------------|-----|
| XMP/EXPO disabled | RAM running at 2133MHz instead of rated speed | Enable XMP in BIOS |
| CPU Core Voltage too low | Instability under load, BSoDs | Increase slightly or set to Auto |
| Fast Boot enabled | Can cause issues with some devices | Disable Fast Boot |
| Secure Boot | Can prevent Linux or older OS from booting | Disable if needed |
| CSM enabled | Can conflict with UEFI boot | Disable CSM for modern systems |
| VT-d/IOMMU disabled | Virtualization issues | Enable for Hyper-V or Proxmox |

---

## 9. Advanced Recovery

```powershell
# From WinPE or Recovery Environment:

# Fix corrupted BCD
bootrec /fixmbr
bootrec /fixboot
bootrec /scanos
bootrec /rebuildbcd

# Offline SFC scan
sfc /scannow /offbootdir=C:\ /offwindir=C:\Windows

# Offline DISM repair
DISM /Image:C:\ /Cleanup-Image /RestoreHealth /Source:D:\Sources\install.wim

# Unlock BitLocker from recovery
manage-bde -unlock C: -RecoveryPassword YOUR-RECOVERY-KEY

# Reset Windows without losing files (from Settings)
# Settings -> System -> Recovery -> Reset this PC -> Keep my files
```

---

## Hardware Test Sequence

When a system has unknown issues, run these tests in order:

```
1. Visual inspection - capacitors, connectors, damage
2. CMOS reset - eliminate BIOS corruption
3. Minimum hardware test - motherboard, CPU, 1 RAM, PSU, GPU
4. PSU voltage test - verify all rails under load
5. MemTest86 - minimum 2 full passes
6. CrystalDiskInfo - check SMART for all drives
7. chkdsk - check file system integrity
8. GPU stress test (FurMark) - 15-30 minutes
9. CPU stress test (Prime95) - 30-60 minutes
10. Windows diagnostics - Event Viewer, SFC, DISM
```

---

## Common Issues & Fixes

| Problem | Advanced Diagnostic | Likely Cause | Fix |
|---------|--------------------|--------------|----|
| POST hangs - no beep | POST code card | CPU or RAM | Reseat CPU, test RAM individually |
| BSoD WHEA error | Check voltages | CPU or RAM hardware | Replace faulty component |
| GPU artifacts | FurMark test | VRAM failure | Replace GPU |
| PSU trips under load | Multimeter under load | PSU failing | Replace PSU |
| System unstable - RAM speed | Check XMP in BIOS | XMP disabled | Enable XMP/EXPO |
| Drive shows SMART errors | CrystalDiskInfo | Drive failing | Backup immediately, replace drive |

---

## Notes
- Systematic isolation is the key to advanced troubleshooting - change one thing at a time
- A working spare of each component is invaluable for swap testing
- Hardware failures often develop gradually - a system that "mostly works" is still failing
- Never ignore SMART errors on storage drives - backup immediately
- Post codes and beep codes are BIOS-specific - look up your motherboard's manual

---

## Related Documents
- [Basic Desktop Troubleshooting](basic-desktop-troubleshooting.md)
- [Enhanced Laptop Troubleshooting](enhanced-laptop-troubleshooting.md)
- [Windows Event Viewer](../Windows/windows-event-viewer.md)
