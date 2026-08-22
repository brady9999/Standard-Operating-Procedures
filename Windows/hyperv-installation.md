# Hyper-V Installation & Configuration
> A guide to installing, configuring, and managing virtual machines using Microsoft Hyper-V.

**Category:** Windows  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Hyper-V** | Hyper-V | Microsoft's built-in hypervisor for Windows Server and Windows 10/11 Pro |
| **Hypervisor** | Hypervisor | Software that creates and manages virtual machines on physical hardware |
| **VM** | Virtual Machine | A software-based computer running on top of the hypervisor |
| **Host** | Hyper-V Host | The physical machine running Hyper-V |
| **Guest** | Guest OS | The operating system running inside a VM |
| **VHD** | Virtual Hard Disk | A file that acts as a hard drive for a VM (.vhd format) |
| **VHDX** | Virtual Hard Disk Extended | The newer, larger, more resilient version of VHD (.vhdx format) |
| **Snapshot** | Checkpoint | A saved state of a VM at a specific point in time |
| **Checkpoint** | Checkpoint | Microsoft's term for what VMware calls a snapshot |
| **Virtual Switch** | vSwitch | A software-based network switch connecting VMs to each other or the network |
| **External Switch** | External vSwitch | Connects VMs to the physical network |
| **Internal Switch** | Internal vSwitch | Connects VMs to each other and to the host but not the external network |
| **Private Switch** | Private vSwitch | Connects VMs only to each other — no host or external access |
| **Generation 1** | Gen 1 VM | Older VM type — BIOS-based, broader OS compatibility |
| **Generation 2** | Gen 2 VM | Newer VM type — UEFI-based, better performance, Secure Boot support |
| **Dynamic Memory** | Dynamic Memory | Allows Hyper-V to automatically adjust RAM allocated to a VM |
| **Live Migration** | Live Migration | Moving a running VM from one host to another with no downtime |

---

## Overview
Hyper-V is Microsoft's built-in hypervisor — it lets you run multiple virtual machines on a single physical server. Each VM behaves like a completely independent computer with its own OS, storage, and network connection, while sharing the physical hardware.

**When to use Hyper-V:**
- Running multiple servers on one physical machine
- Testing software without affecting production systems
- Development and lab environments
- Running legacy operating systems alongside modern ones
- Disaster recovery and backup

---

## Prerequisites

**For Windows Server:**
- Any edition of Windows Server 2016, 2019, or 2022
- 64-bit processor with Second Level Address Translation (SLAT)
- Minimum 4GB RAM (more is better — VMs share the host RAM)
- Virtualization enabled in BIOS/UEFI

**For Windows 10/11:**
- Windows 10/11 Pro, Enterprise, or Education (not Home)
- 64-bit processor with SLAT
- Minimum 4GB RAM
- Virtualization enabled in BIOS/UEFI

### Checking if Virtualization is Enabled
```powershell
# Check if virtualization is enabled
systeminfo | findstr /i "Hyper-V Requirements"

# Or check via Task Manager → Performance → CPU → Virtualization: Enabled
```

---

## 1. Installing Hyper-V

### GUI — Windows Server — Step by Step
1. Open **Server Manager**
2. Click **Manage** → **Add Roles and Features**
3. Click **Next** until **Server Roles**
4. Check **Hyper-V**
5. Click **Add Features** when prompted
6. Click **Next** through:
   - **Virtual Switches** — select the network adapter to use for external access
   - **Migration** — leave defaults
   - **Default Stores** — set where VMs and VHDs are stored
7. Click **Install**
8. **Restart the server** when prompted — required for Hyper-V to load

### GUI — Windows 10/11 — Step by Step
1. Click **Start** → search **Turn Windows features on or off**
2. Scroll to **Hyper-V**
3. Check the box — expand and check all sub-items:
   - Hyper-V Management Tools
   - Hyper-V Platform
4. Click **OK**
5. Windows installs the feature
6. Click **Restart now**

### PowerShell — Windows Server
```powershell
# Install Hyper-V role
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart

# Verify installation
Get-WindowsFeature -Name Hyper-V
```

### PowerShell — Windows 10/11
```powershell
# Enable Hyper-V
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All

# Restart after installation
Restart-Computer
```

---

## How to Open Hyper-V Manager

**Method 1 — Start Menu:**
1. Search **Hyper-V Manager**
2. Click to open

**Method 2 — Server Manager:**
1. Click **Tools** → **Hyper-V Manager**

**Method 3 — Run Dialog:**
1. Press **Windows + R**
2. Type `virtmgmt.msc`
3. Press **Enter**

---

## 2. Creating a Virtual Switch

A virtual switch must be created before VMs can access the network.

### GUI — Step by Step
1. Open **Hyper-V Manager**
2. In the right panel click **Virtual Switch Manager**
3. Select switch type:
   - **External** — VMs can access the physical network and internet
   - **Internal** — VMs can talk to each other and the host only
   - **Private** — VMs can only talk to each other
4. Click **Create Virtual Switch**
5. Give it a name e.g. `External Network`
6. For External — select the physical network adapter to bind to
7. Click **Apply** → **OK**

### PowerShell
```powershell
# Create an external virtual switch
New-VMSwitch -Name "External Network" `
  -NetAdapterName "Ethernet" `
  -AllowManagementOS $true

# Create an internal switch
New-VMSwitch -Name "Internal Network" -SwitchType Internal

# Create a private switch
New-VMSwitch -Name "Private Network" -SwitchType Private

# View all switches
Get-VMSwitch
```

---

## 3. Creating a Virtual Machine

### GUI — Step by Step
1. Open **Hyper-V Manager**
2. In the right panel click **New** → **Virtual Machine**
3. The **New Virtual Machine Wizard** opens → **Next**
4. Enter a **Name** e.g. `Windows-Server-2022` → set location → **Next**
5. Select **Generation**:
   - **Generation 1** — for older OS or maximum compatibility
   - **Generation 2** — for Windows 8/Server 2012 and newer, Linux (recommended)
6. Set **Startup Memory** e.g. `2048` MB (2GB)
   - Check **Use Dynamic Memory** to let Hyper-V adjust automatically
7. Configure networking — select your virtual switch → **Next**
8. Create a virtual hard disk:
   - Set name, location, and size e.g. `80` GB
9. Select installation option:
   - **Install an operating system from a bootable image file** → browse to your ISO
10. Click **Next** → **Finish**
11. The VM appears in the left panel — double-click to open the console → **Start**

### PowerShell
```powershell
# Create a new VM
New-VM -Name "Windows-Server-2022" `
  -Generation 2 `
  -MemoryStartupBytes 2GB `
  -NewVHDPath "C:\VMs\WS2022\WS2022.vhdx" `
  -NewVHDSizeBytes 80GB `
  -SwitchName "External Network"

# Add an ISO for installation
Add-VMDvdDrive -VMName "Windows-Server-2022" `
  -Path "C:\ISOs\windows-server-2022.iso"

# Enable Dynamic Memory
Set-VMMemory -VMName "Windows-Server-2022" `
  -DynamicMemoryEnabled $true `
  -MinimumBytes 512MB `
  -MaximumBytes 4GB

# Start the VM
Start-VM -Name "Windows-Server-2022"
```

---

## 4. Managing VM Power States

### GUI
- Right-click the VM in Hyper-V Manager
- Options: **Start**, **Shut Down**, **Turn Off**, **Save**, **Pause**, **Reset**

> **Shut Down** = graceful shutdown (OS shuts down properly)
> **Turn Off** = hard power off (like pulling the plug — can cause data loss)
> **Save** = saves state to disk and stops the VM (like hibernation)

### PowerShell
```powershell
# Start a VM
Start-VM -Name "Windows-Server-2022"

# Graceful shutdown
Stop-VM -Name "Windows-Server-2022"

# Hard power off
Stop-VM -Name "Windows-Server-2022" -Force

# Save state
Save-VM -Name "Windows-Server-2022"

# Pause
Suspend-VM -Name "Windows-Server-2022"

# Resume
Resume-VM -Name "Windows-Server-2022"

# Restart
Restart-VM -Name "Windows-Server-2022"
```

---

## 5. Taking and Managing Checkpoints

Checkpoints save the current state of a VM so you can restore it if something goes wrong.

### GUI — Step by Step
1. Right-click the VM → **Checkpoint**
2. A checkpoint appears in the Checkpoints section of Hyper-V Manager
3. To restore — right-click the checkpoint → **Apply**
4. To delete — right-click → **Delete Checkpoint**

> ⚠️ **Warning:** Checkpoints consume significant disk space. Don't leave them long-term in production — they degrade performance over time.

### PowerShell
```powershell
# Create a checkpoint
Checkpoint-VM -Name "Windows-Server-2022" -SnapshotName "Before Update"

# View all checkpoints for a VM
Get-VMCheckpoint -VMName "Windows-Server-2022"

# Restore a checkpoint
Restore-VMCheckpoint -Name "Before Update" -VMName "Windows-Server-2022" -Confirm:$false

# Delete a checkpoint
Remove-VMCheckpoint -VMName "Windows-Server-2022" -Name "Before Update"
```

---

## 6. Modifying VM Hardware

### GUI — Step by Step
1. The VM must be **off** to change most hardware settings
2. Right-click the VM → **Settings**
3. In the left panel select the component to modify:
   - **Memory** — change RAM, enable/disable dynamic memory
   - **Processor** — change virtual CPU count
   - **Hard Drive** — add/remove/resize disks
   - **Network Adapter** — change virtual switch
   - **DVD Drive** — attach/detach ISOs
4. Make changes → **Apply** → **OK**

### PowerShell
```powershell
# Change RAM
Set-VM -Name "Windows-Server-2022" -MemoryStartupBytes 4GB

# Change CPU count
Set-VMProcessor -VMName "Windows-Server-2022" -Count 4

# Add a new virtual disk
Add-VMHardDiskDrive -VMName "Windows-Server-2022" `
  -NewVHDPath "C:\VMs\WS2022\DataDisk.vhdx" `
  -NewVHDSizeBytes 50GB

# Connect to a different switch
Connect-VMNetworkAdapter -VMName "Windows-Server-2022" -SwitchName "Internal Network"
```

---

## 7. Connecting to a VM Console

### GUI — Step by Step
1. Double-click the VM in Hyper-V Manager
2. The **Virtual Machine Connection** window opens
3. Click **Start** if the VM is off
4. Interact with the VM through the window

### PowerShell / Command Line
```powershell
# Open VM console
vmconnect localhost "Windows-Server-2022"
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Hyper-V won't install | Virtualization disabled in BIOS | Enable Intel VT-x or AMD-V in BIOS/UEFI settings |
| VM won't start | Insufficient RAM or disk space | Free up resources on the host |
| No network in VM | Wrong virtual switch selected | Check VM network adapter settings |
| VM running slow | Host overloaded or too many VMs | Reduce number of running VMs or add more RAM to host |
| Can't resize VHD | VM is running or has checkpoints | Shut down VM and delete checkpoints first |
| Black screen in console | Display driver issue | Install Hyper-V Integration Services in the guest OS |

---

## Quick Reference

```powershell
# List all VMs
Get-VM

# List running VMs
Get-VM | Where-Object {$_.State -eq "Running"}

# Start all VMs
Get-VM | Start-VM

# Stop all VMs gracefully
Get-VM | Stop-VM

# View VM resource usage
Get-VM | Select-Object Name, State, CPUUsage, MemoryAssigned

# List all virtual switches
Get-VMSwitch

# List all checkpoints
Get-VMCheckpoint -VMName "VM-Name"

# Export a VM
Export-VM -Name "Windows-Server-2022" -Path "D:\VMBackup"

# Import a VM
Import-VM -Path "D:\VMBackup\Windows-Server-2022\Virtual Machines\*.vmcx"
```

---

## Notes
- Always install **Hyper-V Integration Services** in guest VMs for best performance — Windows VMs install these automatically, Linux VMs need `linux-image-extra` or equivalent
- Use **Generation 2** VMs for any OS that supports it — better performance and Secure Boot
- VHDX format is preferred over VHD — larger max size (64TB vs 2TB), better resilience
- Checkpoints are useful for testing but should not be used as a backup strategy
- For production servers use **Hyper-V Replica** or a dedicated backup solution
- Dynamic Memory is great for development environments but can cause issues in production — test carefully

---

## Related Documents
- [Virtual Machine Routing](../Virtual-Machines/virtual-machine-routing.md)
- [Virtual Machine Backup and Restore](../Virtual-Machines/virtual-machine-backup-and-restore.md)
- [DNS Configuration](dns-configuration.md)
