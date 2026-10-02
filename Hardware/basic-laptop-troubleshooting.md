# Basic Laptop Troubleshooting
> A guide to diagnosing and resolving common laptop hardware and software issues.

**Category:** Hardware  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **POST** | Power-On Self-Test | The diagnostic test a computer runs when it first powers on |
| **BIOS/UEFI** | Basic Input/Output System / Unified Extensible Firmware Interface | The firmware that initializes hardware before the OS loads |
| **Safe Mode** | Safe Mode | A Windows diagnostic mode that loads only essential drivers |
| **Driver** | Device Driver | Software that allows the OS to communicate with hardware |
| **RAM** | Random Access Memory | Short-term memory used by running programs |
| **HDD/SSD** | Hard Disk Drive / Solid State Drive | Long-term storage for the OS and files |
| **Thermal Paste** | Thermal Interface Material | A compound between the CPU and heatsink to transfer heat |
| **BSoD** | Blue Screen of Death | A Windows critical error that causes a system crash |
| **Hibernation** | Hibernate | Saves RAM to disk and powers off - faster than full shutdown |
| **AC Adapter** | AC Adapter | The external power supply (charger) for a laptop |

---

## Overview
Laptop troubleshooting follows a logical process - start with the simplest possible cause and work toward the more complex. Most issues fall into a few common categories: power, display, connectivity, performance, and software.

**Golden Rule:** Before doing anything else - restart the laptop. This resolves a surprising number of issues.

---

## Troubleshooting Methodology

```
1. IDENTIFY - What exactly is the problem? Get specific symptoms.
2. REPRODUCE - Can you make the problem happen again?
3. ISOLATE - Is it hardware or software? Is it one app or everything?
4. RESEARCH - Search the error message or symptoms.
5. FIX - Apply the solution.
6. VERIFY - Confirm the problem is resolved.
7. DOCUMENT - Record what the problem was and how it was fixed.
```

---

## 1. Laptop Won't Turn On

### Step by Step
1. Check the power adapter is plugged into both the laptop and the wall outlet
2. Try a different wall outlet
3. Check the power adapter LED - is it lit? If not the adapter may be faulty
4. Remove the battery (if removable), hold the power button for 30 seconds, reinsert battery and try again
5. Try powering on with only AC adapter - no battery
6. Listen and watch for any signs of life:
   - Fan spinning?
   - LEDs flickering?
   - Screen briefly lit?
7. If nothing - suspect power adapter, battery, or motherboard failure

### Common Causes and Fixes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| No lights, no fan, no response | Dead battery or faulty adapter | Test with known-good adapter |
| Fan spins but no display | RAM or display issue | See display section below |
| Powers on briefly then off | Thermal shutdown or RAM issue | Check ventilation, reseat RAM |
| Battery charges but won't run on battery | Faulty battery | Replace battery |

---

## 2. Display Issues

### Screen Is Black / No Display

1. Check if the laptop is actually on - listen for fan, look for keyboard backlighting
2. Press **Fn + F7** (or the display toggle key on your model) to cycle through display modes
3. Connect an external monitor - if external works, the issue is the laptop screen or cable
4. Shine a flashlight at the screen at an angle - if you can see a faint image the backlight has failed
5. Try pressing **Windows + P** to change display mode

### Screen Flickering
- Update or roll back the display driver
- Check display cable connection (may require disassembly)
- Test refresh rate: **Display Settings -> Advanced Display -> Refresh Rate**

### Dead Pixels
- Run a dead pixel test (solid color full screen)
- Isolated dead pixels - cosmetic issue only
- Large areas of dead pixels - screen replacement required

### PowerShell - Display Diagnostics
```powershell
# View display adapter info
Get-WmiObject Win32_VideoController | Select-Object Name, DriverVersion, Status

# Check display resolution
Get-WmiObject Win32_VideoController | Select-Object CurrentHorizontalResolution, CurrentVerticalResolution

# Update display driver via Windows Update
Get-WindowsUpdate -Category "Drivers" | Install-WindowsUpdate
```

---

## 3. Battery Issues

### Battery Not Charging
1. Try a different outlet
2. Check the charging port for debris or damage
3. Inspect the charging cable for damage
4. Check Battery Health in Windows

```powershell
# Generate battery report
powercfg /batteryreport /output "C:\battery-report.html"
# Open the HTML file to view battery health and capacity history
```

### Battery Drains Quickly
1. Check what's consuming power:
   - **Task Manager** -> **More Details** -> sort by CPU or Power
2. Reduce screen brightness
3. Enable battery saver mode
4. Disable Bluetooth and WiFi when not needed
5. Check for background apps

```powershell
# View power settings
powercfg /query

# Generate energy report (detailed power analysis)
powercfg /energy /output "C:\energy-report.html"

# Check sleep states
powercfg /sleepstudy
```

---

## 4. Performance Issues

### Laptop Is Slow

**Quick checks:**
1. Restart the laptop
2. Check Task Manager for high CPU, RAM, or disk usage
3. Check available disk space - Windows needs at least 10-15% free

```powershell
# Check CPU usage
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, CPU

# Check RAM usage
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10 Name, WorkingSet

# Check disk space
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Used, Free

# Check disk health
Get-PhysicalDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus

# Run disk cleanup
cleanmgr /sagerun:1

# Check startup programs
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location
```

### Overheating
1. Check that vents are not blocked
2. Use the laptop on a hard flat surface - not a bed or carpet
3. Clean vents with compressed air
4. Check CPU temperature

```powershell
# Check CPU temperature (requires third-party tool or WMI)
Get-WmiObject MSAcpi_ThermalZoneTemperature -Namespace root/wmi |
  Select-Object @{n='Temperature';e={($_.CurrentTemperature / 10) - 273.15}}
```

---

## 5. Wi-Fi Issues

### Can't Connect to Wi-Fi

1. Check if Wi-Fi is enabled (Fn + Wi-Fi key or airplane mode toggle)
2. Forget the network and reconnect
3. Restart the router and laptop
4. Run the network troubleshooter

```powershell
# View Wi-Fi adapter status
Get-NetAdapter | Where-Object MediaType -eq "802.11"

# View available Wi-Fi networks
netsh wlan show networks

# View current Wi-Fi connection
netsh wlan show interfaces

# Disable and re-enable Wi-Fi adapter
Disable-NetAdapter -Name "Wi-Fi" -Confirm:$false
Enable-NetAdapter -Name "Wi-Fi" -Confirm:$false

# Reset TCP/IP stack
netsh int ip reset
netsh winsock reset

# Release and renew IP address
ipconfig /release
ipconfig /renew

# Flush DNS cache
ipconfig /flushdns

# Forget a saved Wi-Fi network
netsh wlan delete profile name="NetworkName"
```

---

## 6. Audio Issues

### No Sound
1. Check volume is not muted
2. Check the correct playback device is selected
3. Right-click the speaker icon -> **Open Sound Settings**
4. Test with headphones to determine if it's speakers or software

```powershell
# List audio devices
Get-WmiObject Win32_SoundDevice | Select-Object Name, Status

# Check audio service
Get-Service AudioSrv | Select-Object Name, Status
Start-Service AudioSrv
```

---

## 7. Keyboard and Touchpad Issues

### Keys Not Working
1. Restart the laptop
2. Check for debris under the keys
3. Test in a different application
4. Try an external USB keyboard

### Touchpad Not Working
1. Check if touchpad is disabled - many laptops have **Fn + F9** (or similar) to toggle it
2. Check in **Device Manager** that the touchpad driver is installed

```powershell
# Check touchpad/keyboard in Device Manager
Get-PnpDevice | Where-Object {$_.Class -eq "HIDClass"} | Select-Object FriendlyName, Status

# Reinstall touchpad driver
Get-PnpDevice -FriendlyName "*touchpad*" | Disable-PnpDevice -Confirm:$false
Get-PnpDevice -FriendlyName "*touchpad*" | Enable-PnpDevice -Confirm:$false
```

---

## 8. Blue Screen of Death (BSoD)

### Immediate Steps
1. Note the error code on screen e.g. `DRIVER_IRQL_NOT_LESS_OR_EQUAL`
2. Let the system restart - Windows automatically creates a dump file
3. Check Event Viewer -> **Windows Logs** -> **System** for critical errors

```powershell
# Read minidump files (requires WinDbg or third-party tool)
# Get recent BSoD events from Event Viewer
Get-WinEvent -FilterHashtable @{LogName='System'; Level=1} |
  Select-Object TimeCreated, Message -First 10

# Check reliability monitor
Get-WinEvent -LogName "Microsoft-Windows-Reliability-Analysis" -ErrorAction SilentlyContinue

# Run system file checker
sfc /scannow

# Run DISM to repair Windows image
DISM /Online /Cleanup-Image /RestoreHealth
```

### Common BSoD Codes

| Error Code | Likely Cause |
|-----------|-------------|
| DRIVER_IRQL_NOT_LESS_OR_EQUAL | Faulty driver |
| MEMORY_MANAGEMENT | RAM issue |
| KERNEL_SECURITY_CHECK_FAILURE | Driver or RAM issue |
| CRITICAL_PROCESS_DIED | Corrupted system files |
| NTFS_FILE_SYSTEM | Disk issue |
| PAGE_FAULT_IN_NONPAGED_AREA | RAM or driver issue |

---

## 9. Common Fixes Quick Reference

```powershell
# System File Checker - repairs corrupted Windows files
sfc /scannow

# DISM - repairs Windows image
DISM /Online /Cleanup-Image /RestoreHealth

# Check disk for errors
chkdsk C: /f /r

# Reset network settings
netsh int ip reset
netsh winsock reset
ipconfig /flushdns

# Battery report
powercfg /batteryreport /output "C:\battery-report.html"

# Generate system info report
msinfo32 /report "C:\system-info.txt"

# Check startup impact
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command

# View device errors
Get-PnpDevice | Where-Object Status -ne "OK" | Select-Object FriendlyName, Status, Problem
```

---

## Common Issues & Fixes Summary

| Problem | First Step | Second Step | Escalate If |
|---------|-----------|-------------|-------------|
| Won't turn on | Check power adapter | Remove battery, hold power 30s | No response at all |
| Black screen | Toggle display (Fn+F7) | Connect external monitor | External also fails |
| Slow performance | Restart, check Task Manager | Run disk cleanup, check for malware | Hardware failing |
| Wi-Fi not connecting | Toggle Wi-Fi, restart | Reset TCP/IP | Driver missing |
| BSoD | Note error code, restart | Run sfc /scannow | Recurring BSoDs |
| Battery not charging | Try different outlet | Generate battery report | Charging port damaged |

---

## Notes
- Always ask the user "what changed recently?" - new software, updates, or physical damage often explain the problem
- Document the exact error message - searching it online usually leads directly to the solution
- Never skip the restart - it solves more problems than any other step
- Check the manufacturer's support site for model-specific issues and driver downloads
- If under warranty - contact the manufacturer before opening the device

---

## Related Documents
- [Enhanced Laptop Troubleshooting](enhanced-laptop-troubleshooting.md)
- [Basic Desktop Troubleshooting](basic-desktop-troubleshooting.md)
- [Windows Event Viewer](../Windows/windows-event-viewer.md)
