# Peripheral Device Troubleshooting
> A guide to diagnosing and resolving issues with monitors, keyboards, mice, USB devices, webcams, headsets, and other peripherals.

**Category:** Hardware  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **HID** | Human Interface Device | A class of USB devices including keyboards, mice, and game controllers |
| **USB Hub** | USB Hub | A device that expands one USB port into multiple ports |
| **Powered Hub** | Powered USB Hub | A USB hub with its own power supply — better for power-hungry devices |
| **Driver** | Device Driver | Software allowing the OS to communicate with the device |
| **Plug and Play** | Plug and Play | Technology that automatically detects and installs drivers for new devices |
| **Device Manager** | Device Manager | Windows tool for viewing and managing all connected hardware |
| **DPI** | Dots Per Inch | Sensitivity setting for mice — higher DPI = faster cursor movement |
| **Hz / Refresh Rate** | Refresh Rate | How many times per second a monitor updates — higher is smoother |
| **EDID** | Extended Display Identification Data | Data a monitor sends to the GPU describing its capabilities |
| **KVM** | Keyboard, Video, Mouse Switch | A device allowing one keyboard/monitor/mouse to control multiple computers |

---

## Overview
Peripheral issues are usually caused by one of three things:
1. **Physical connection** — cable, port, or device damage
2. **Driver** — missing, corrupted, or outdated software
3. **Settings** — wrong configuration in Windows or the device's software

Always try the device on a different computer first — this immediately tells you if the problem is the device or the system.

---

## 1. Monitor Issues

### No Signal / Black Screen
```
1. Check the monitor power cable is connected and monitor is on
2. Verify the correct input source is selected (HDMI 1, DisplayPort, etc.)
3. Try a different cable (HDMI, DisplayPort, DVI)
4. Try a different port on the GPU
5. Try the monitor on a different computer
6. Connect directly to GPU — not through a KVM or hub
```

### Monitor Showing Wrong Resolution
```powershell
# View current display settings
Get-WmiObject Win32_VideoController |
  Select-Object Name, CurrentHorizontalResolution, CurrentVerticalResolution, CurrentRefreshRate

# Set resolution via Display Settings
# Right-click desktop → Display Settings → Resolution → select correct resolution
```

### Monitor Flickering
```powershell
# Check refresh rate
Get-WmiObject Win32_VideoController | Select-Object CurrentRefreshRate

# Update display driver
Get-WmiObject Win32_VideoController | Select-Object DriverVersion, DriverDate
```

**Common causes:**
- Wrong refresh rate set (try 60Hz if flickering at 75Hz+)
- Faulty cable — try a new cable
- Loose cable connection — reseat firmly
- Driver issue — update or roll back display driver
- Monitor hardware failure

### Dead Pixels
- Run a dead pixel test — display solid red, green, blue, white, and black full screen
- Single dead pixel — usually cosmetic and covered by warranty if within threshold
- Large cluster or line of dead pixels — panel failure, replacement needed

### Color Issues
```powershell
# Check ICC color profile
Get-WmiObject Win32_VideoController | Select-Object Name

# Reset color calibration
# Control Panel → Color Management → All Profiles → Reset
```

---

## 2. Keyboard Issues

### Keys Not Responding

```powershell
# Check keyboard in Device Manager
Get-PnpDevice | Where-Object {$_.Class -eq "Keyboard"} | Select-Object FriendlyName, Status

# View HID devices
Get-PnpDevice | Where-Object {$_.Class -eq "HIDClass"} | Select-Object FriendlyName, Status

# Reinstall keyboard driver
Get-PnpDevice -FriendlyName "*keyboard*" | Disable-PnpDevice -Confirm:$false
Start-Sleep -Seconds 3
Get-PnpDevice -FriendlyName "*keyboard*" | Enable-PnpDevice -Confirm:$false
```

**Physical checks:**
1. Try a different USB port
2. Try the keyboard on another computer
3. Check for debris under keys
4. For wireless keyboards — replace batteries or check USB receiver

### Keyboard Typing Wrong Characters
- Check the keyboard language/layout in Settings
- **Settings → Time & Language → Language → Keyboard**
- Press **Windows + Space** to switch between input languages
- Check for sticky keys: **Settings → Ease of Access → Keyboard**

### Num Lock / Caps Lock Issues
```powershell
# Check current keyboard state
Add-Type -AssemblyName System.Windows.Forms
[System.Windows.Forms.Control]::IsKeyLocked([System.Windows.Forms.Keys]::CapsLock)
[System.Windows.Forms.Control]::IsKeyLocked([System.Windows.Forms.Keys]::NumLock)
```

---

## 3. Mouse Issues

### Mouse Not Moving / Detected

```powershell
# View mouse devices
Get-PnpDevice | Where-Object {$_.Class -eq "Mouse"} | Select-Object FriendlyName, Status

# Reinstall mouse driver
Get-PnpDevice -FriendlyName "*mouse*" | Disable-PnpDevice -Confirm:$false
Start-Sleep -Seconds 3
Get-PnpDevice -FriendlyName "*mouse*" | Enable-PnpDevice -Confirm:$false
```

**Physical checks:**
1. Try a different USB port
2. Try the mouse on another computer
3. Check mouse sensor — clean the bottom optical sensor
4. For wireless mice — replace batteries, re-pair USB receiver

### Mouse Cursor Jumping or Erratic
- Clean the mouse pad or surface
- Try a different surface (avoid glass)
- Clean the optical sensor with compressed air
- Update mouse driver/firmware

### Mouse Buttons Not Working
- Test in a different application
- Check if mouse software is conflicting
- Try a different USB port
- Test on another computer

---

## 4. USB Device Issues

### Device Not Recognized

```powershell
# View all USB devices
Get-PnpDevice | Where-Object {$_.InstanceId -like "USB*"} | Select-Object FriendlyName, Status

# View devices with issues
Get-PnpDevice | Where-Object {$_.Status -ne "OK"} | Select-Object FriendlyName, Status, Problem

# Reset USB controllers
# Device Manager → Universal Serial Bus Controllers → right-click each Root Hub → Uninstall
# Then scan for hardware changes

# View USB event log
Get-WinEvent -LogName "Microsoft-Windows-DriverFrameworks-UserMode/Operational" -MaxEvents 50 |
  Where-Object {$_.Message -like "*USB*"} | Select-Object TimeCreated, Message
```

**Troubleshooting steps:**
1. Try a different USB port
2. Try a different USB cable
3. Test the device on another computer
4. Try connecting directly to the PC — not through a hub
5. Use a powered USB hub if the device needs more power
6. Check Device Manager for errors
7. Reinstall the device driver

### USB Selective Suspend (Power Management Issues)
Windows may power down USB ports to save energy — causing devices to disconnect.

```powershell
# Disable USB selective suspend
powercfg -setacvalueindex SCHEME_CURRENT 2a737441-1930-4402-8d77-b2bebba308a3 48e6b7a6-50f5-4782-a5d4-53bb8f07e226 0
powercfg -setactive SCHEME_CURRENT

# Or via GUI:
# Power Options → Change plan settings → Change advanced power settings
# USB Settings → USB selective suspend setting → Disabled
```

---

## 5. Webcam Issues

### Webcam Not Detected

```powershell
# View webcam devices
Get-PnpDevice | Where-Object {$_.Class -eq "Camera" -or $_.FriendlyName -like "*webcam*"} |
  Select-Object FriendlyName, Status

# Check camera privacy settings
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\CapabilityAccessManager\ConsentStore\webcam" |
  Select-Object Value
```

### App Can't Access Webcam
1. **Settings → Privacy & Security → Camera**
2. Ensure "Camera access" is ON
3. Ensure the specific app has camera permission
4. Close other apps that might be using the camera

### Poor Image Quality
- Clean the webcam lens
- Check lighting — face a light source, don't have a bright window behind you
- Update webcam driver
- Adjust camera settings in the webcam software

---

## 6. Headset / Audio Device Issues

### Headset Not Detected

```powershell
# View audio devices
Get-WmiObject Win32_SoundDevice | Select-Object Name, Status

# View audio endpoints
Get-AudioDevice -List  # Requires AudioDeviceCmdlets module

# Restart audio services
Restart-Service AudioSrv
Restart-Service AudioEndpointBuilder
```

**Physical checks:**
1. Check the headset is fully plugged in
2. Try a different USB port (for USB headsets)
3. For 3.5mm headsets — check the correct jack (green = audio, pink = microphone)
4. Combo jack headsets need a single 4-pole jack

### Setting Default Audio Device

```powershell
# GUI: Right-click speaker icon → Open Sound Settings → Choose output/input device

# Via PowerShell (requires AudioDeviceCmdlets)
Install-Module -Name AudioDeviceCmdlets
Get-AudioDevice -List
Set-AudioDevice -Index 2  # Set by index number
```

### Microphone Not Working
1. **Settings → System → Sound → Input** — verify mic is selected
2. Check microphone privacy settings
3. Check mic volume is not zero
4. Test in Windows Voice Recorder app

---

## 7. Barcode Scanners and Specialty Devices

Barcode scanners typically appear as keyboards to Windows — they send keystrokes when scanning.

**Common issues:**
- Wrong character encoding — check scanner configuration for the character set
- Missing prefix/suffix characters — configure in scanner settings
- Slow scan speed — check USB polling rate
- Not recognized — try different USB port, check HID driver

```powershell
# View HID devices (scanners appear here)
Get-PnpDevice | Where-Object {$_.Class -eq "HIDClass"} | Select-Object FriendlyName, Status
```

---

## 8. Common Device Manager Error Codes

| Code | Meaning | Fix |
|------|---------|-----|
| **Code 10** | Device cannot start | Update or reinstall driver |
| **Code 18** | Device drivers need reinstalling | Reinstall driver |
| **Code 19** | Registry error | Run registry repair or reinstall driver |
| **Code 28** | Drivers not installed | Install correct driver |
| **Code 43** | Device stopped — Windows reported an error | Reinstall driver, check hardware |
| **Code 45** | Device not connected | Reconnect device |

```powershell
# Find devices with errors by code
Get-WmiObject Win32_PnPEntity | Where-Object {$_.ConfigManagerErrorCode -ne 0} |
  Select-Object Name, ConfigManagerErrorCode, Status
```

---

## 9. Quick Diagnostic Steps for Any Peripheral

```
Step 1: Try a different port (USB port, video port, audio jack)
Step 2: Try a different cable
Step 3: Try the device on a different computer
Step 4: Check Device Manager for errors
Step 5: Uninstall and reinstall the driver
Step 6: Download the latest driver from the manufacturer
Step 7: Try a different USB hub or connect directly
Step 8: Check Windows privacy settings (camera, microphone)
Step 9: Restart the relevant Windows service
Step 10: Restart the computer
```

---

## Common PowerShell Reference

```powershell
# View all connected devices
Get-PnpDevice | Select-Object FriendlyName, Class, Status

# Find devices with problems
Get-PnpDevice | Where-Object Status -ne "OK"

# Disable and re-enable a device
Disable-PnpDevice -FriendlyName "Device Name" -Confirm:$false
Enable-PnpDevice -FriendlyName "Device Name" -Confirm:$false

# View USB devices
Get-PnpDevice | Where-Object {$_.InstanceId -like "USB*"}

# View audio devices
Get-WmiObject Win32_SoundDevice

# Restart audio
Restart-Service AudioSrv, AudioEndpointBuilder

# View display adapters
Get-WmiObject Win32_VideoController
```

---

## Notes
- Most peripheral issues are solved by trying the device on another computer first — this isolates hardware vs system issues
- USB 3.0 ports are blue — use these for high-bandwidth devices (webcams, external drives)
- Avoid USB hubs for power-hungry devices — use direct port connections or powered hubs
- Always download drivers from the manufacturer's website — not from third-party driver sites
- Generic Windows drivers work for basic functions — manufacturer drivers unlock full features

---

## Related Documents
- [Printer Troubleshooting](printer-troubleshooting.md)
- [Basic Desktop Troubleshooting](basic-desktop-troubleshooting.md)
- [Basic Laptop Troubleshooting](basic-laptop-troubleshooting.md)
