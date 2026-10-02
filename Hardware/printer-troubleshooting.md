# Printer Troubleshooting
> A guide to diagnosing and resolving common printer issues including connectivity, print quality, and driver problems.

**Category:** Hardware  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Print Spooler** | Print Spooler Service | Windows service that manages print jobs in a queue |
| **Print Queue** | Print Queue | A list of documents waiting to be printed |
| **Driver** | Printer Driver | Software allowing Windows to communicate with the printer |
| **PCL** | Printer Command Language | A common printer language developed by HP |
| **PostScript** | PostScript | A page description language used by many professional printers |
| **Duplex** | Duplex Printing | Printing on both sides of a page automatically |
| **Toner** | Toner | The powder used in laser printers to form text and images |
| **Ink Cartridge** | Ink Cartridge | The replaceable container of ink in inkjet printers |
| **Fuser** | Fuser Unit | The component in a laser printer that melts toner onto paper |
| **Drum** | Imaging Drum | The photosensitive cylinder in a laser printer |
| **Network Printer** | Network Printer | A printer connected to the network - shared by multiple users |
| **Local Printer** | Local Printer | A printer connected directly to a computer via USB |
| **IPP** | Internet Printing Protocol | A protocol for network printing |
| **SMB** | Server Message Block | The protocol used for Windows shared printers |
| **SNMP** | Simple Network Management Protocol | Used to monitor printer status remotely |

---

## Overview
Printer issues fall into four main categories:
- **Connectivity** - can't find or connect to the printer
- **Driver** - Windows can't communicate properly with the printer
- **Print Quality** - output looks wrong
- **Hardware** - physical component failure

Always start with the simplest fix and work toward hardware replacement.

---

## 1. Printer Not Detected / Can't Print

### USB Printer
```
1. Check USB cable is connected at both ends
2. Try a different USB port on the computer
3. Try a different USB cable
4. Power cycle the printer (off -> wait 30s -> on)
5. Check Device Manager for printer with errors
6. Reinstall the driver
```

### Network Printer
```
1. Check the printer's IP address (print a config page or check the display)
2. Ping the printer IP from the computer
3. Check the printer is on the correct VLAN
4. Verify the computer and printer are on the same network
5. Check firewall is not blocking printer communication
6. Try connecting directly via IP address
```

```powershell
# Find printers on the network
Get-Printer | Select-Object Name, DriverName, PortName, PrinterStatus

# Ping printer
Test-NetConnection -ComputerName 192.168.1.50

# Add network printer by IP
Add-Printer -Name "Office Laser" -DriverName "HP LaserJet" -PortName "192.168.1.50"

# Add printer port
Add-PrinterPort -Name "192.168.1.50" -PrinterHostAddress "192.168.1.50"

# View printer ports
Get-PrinterPort | Select-Object Name, PrinterHostAddress

# Remove and re-add printer
Remove-Printer -Name "Office Laser"
```

---

## 2. Print Spooler Issues

The Print Spooler service manages all print jobs. When it crashes or gets stuck all printing stops.

### Restarting the Print Spooler

```powershell
# Stop spooler
Stop-Service -Name Spooler -Force

# Clear the print queue (delete all stuck jobs)
Remove-Item -Path "C:\Windows\System32\spool\PRINTERS\*" -Force -Recurse

# Start spooler
Start-Service -Name Spooler

# Verify spooler is running
Get-Service Spooler
```

### Clearing a Stuck Print Job

```powershell
# View all print jobs
Get-PrintJob -PrinterName "PrinterName"

# Remove a specific stuck job
Remove-PrintJob -PrinterName "PrinterName" -ID 1

# Remove all print jobs for a printer
Get-PrintJob -PrinterName "PrinterName" | Remove-PrintJob

# Nuclear option - stop spooler, clear all jobs, restart
Stop-Service Spooler -Force
Get-ChildItem "C:\Windows\System32\spool\PRINTERS" | Remove-Item -Force
Start-Service Spooler
```

---

## 3. Driver Issues

### Reinstalling a Printer Driver

```powershell
# View installed printer drivers
Get-PrinterDriver | Select-Object Name, Manufacturer

# Remove a printer
Remove-Printer -Name "Printer Name" -Confirm:$false

# Remove a printer driver
Remove-PrinterDriver -Name "HP LaserJet Pro" -Confirm:$false

# List all installed printers
Get-Printer

# Add a printer with a specific driver
Add-Printer -Name "HP LaserJet" -DriverName "HP LaserJet Universal PCL6" -PortName "192.168.1.50"
```

### GUI - Reinstall Driver Step by Step
1. Open **Devices and Printers** (or Settings -> Printers & Scanners)
2. Right-click the printer -> **Remove device**
3. Open **Device Manager** -> **View** -> **Show hidden devices**
4. Expand **Printers** -> right-click any remaining entries -> **Uninstall device**
5. Open **Print Management** (if available) -> **Drivers** -> remove the driver
6. Restart the Print Spooler
7. Download the latest driver from the manufacturer's website
8. Install the new driver
9. Add the printer again

---

## 4. Print Quality Issues

### Laser Printer Quality Problems

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| Faded output | Low toner | Replace toner cartridge |
| Streaks or lines | Dirty drum or fuser | Clean drum, replace if damaged |
| Smearing or not fusing | Fuser issue | Replace fuser unit |
| Ghosting (faint repeated image) | Drum not clearing | Replace drum unit |
| Black page | Drum exposed to light | Replace drum |
| Spots or dots | Drum damage | Replace drum |
| Wrinkled paper | Fuser temperature issue | Check fuser unit |

### Inkjet Printer Quality Problems

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| Missing colors | Clogged nozzle | Run nozzle cleaning utility |
| Banding (horizontal lines) | Partially clogged nozzle | Run print head cleaning |
| Wrong colors | Empty or wrong cartridge | Replace cartridge |
| Smearing | Wrong paper type or ink not dry | Use correct paper, allow more dry time |
| Streaks | Dirty print head | Clean print head |

### Running Printer Diagnostics
Most printers have a built-in self-test:
- **HP:** Hold **Go** button during power on
- **Epson:** Hold specific button combination (check model manual)
- **Brother:** Navigate to **Reports** -> **Print Settings**
- **Canon:** Navigate to **Settings** -> **Device Settings** -> **Print Status Sheet**

---

## 5. Paper Jams

### Clearing a Paper Jam
1. Turn off the printer
2. Open all access panels (front, back, and top depending on model)
3. Gently pull jammed paper in the direction of the paper path
   - Never pull against the paper path direction - this can damage the rollers
4. Check for torn pieces of paper - even a small piece will cause another jam
5. Close all panels
6. Power on and run a test page

### Frequent Paper Jams

| Cause | Fix |
|-------|-----|
| Wrong paper type or weight | Use paper within the printer's specifications |
| Overfilled paper tray | Don't exceed the max fill line |
| Worn pickup rollers | Clean or replace pickup rollers |
| Damaged paper guide | Adjust or replace paper guides |
| Damp paper | Store paper in a dry location |
| Torn paper remnants | Fully clear any debris from paper path |

---

## 6. Shared Network Printer Issues

### Printer Is Shared from a Server

```powershell
# View shared printers on the server
Get-Printer | Where-Object Shared -eq $True | Select-Object Name, ShareName

# Connect to a shared printer
Add-Printer -ConnectionName "\\Server\PrinterShareName"

# View printers connected from a remote server
Get-Printer -ComputerName "PrintServer"

# Deploy printer via Group Policy
# Computer Configuration -> Preferences -> Control Panel Settings -> Printers -> New -> Shared Printer
```

### Printer Accessible to Some Users But Not Others
```powershell
# Check printer permissions
$printer = Get-Printer -Name "Office Printer" -Full
$printer.PermissionSDDL

# Grant a user print access via GUI
# Devices and Printers -> Right-click printer -> Printer Properties -> Security tab
# Add user/group -> Check Print
```

---

## 7. Common Printer PowerShell Commands

```powershell
# List all printers
Get-Printer

# List print jobs
Get-PrintJob -PrinterName "PrinterName"

# Pause a printer
Suspend-Printer -Name "PrinterName"

# Resume a printer
Resume-Printer -Name "PrinterName"

# Set default printer
(New-Object -ComObject WScript.Network).SetDefaultPrinter("PrinterName")

# Print a test page
$printer = Get-WmiObject -Query "SELECT * FROM Win32_Printer WHERE Name='PrinterName'"
$printer.PrintTestPage()

# View printer status
Get-Printer | Select-Object Name, PrinterStatus, JobCount

# Restart Print Spooler
Restart-Service Spooler

# Clear all print jobs
Stop-Service Spooler -Force
Remove-Item "C:\Windows\System32\spool\PRINTERS\*" -Force
Start-Service Spooler
```

---

## Common Issues & Fixes

| Problem | Quick Fix | If That Fails |
|---------|-----------|---------------|
| Nothing prints | Restart spooler, clear queue | Reinstall driver |
| Stuck print job | Remove job via print queue | Stop spooler, clear spool folder |
| Can't find network printer | Ping printer IP | Check VLAN and firewall |
| Poor print quality | Run cleaning cycle | Replace toner/cartridge |
| Frequent paper jams | Check paper type and fill level | Clean/replace rollers |
| Offline status | Restart printer and spooler | Reinstall driver, check IP |
| Wrong printer selected | Set correct default printer | `(New-Object -ComObject WScript.Network).SetDefaultPrinter()` |

---

## Notes
- The Print Spooler is the first thing to check for any printing issue - restart it before anything else
- Always download drivers from the manufacturer's website - avoid generic drivers where possible
- Network printers should have a static IP or DHCP reservation - a changing IP breaks all connections
- Laser printer consumables (toner, drum, fuser) have page count limits - check the printer's status page
- When replacing toner - remove the protective tape before installing

---

## Related Documents
- [Peripheral Device Troubleshooting](peripheral-device-troubleshooting.md)
- [Windows Event Viewer](../Windows/windows-event-viewer.md)
- [Basic Desktop Troubleshooting](basic-desktop-troubleshooting.md)
