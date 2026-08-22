# File and Storage Server Manager
> A guide to configuring file shares, storage pools, and storage management using Windows Server Manager.

**Category:** Windows  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **SMB** | Server Message Block | The protocol Windows uses to share files over a network |
| **Share** | File Share | A folder made accessible over the network |
| **UNC Path** | Universal Naming Convention Path | The network path to a share e.g. `\\server\share` |
| **NTFS Permissions** | NTFS Permissions | File-level permissions set on files and folders |
| **Share Permissions** | Share Permissions | Network-level permissions controlling who can access the share |
| **Storage Pool** | Storage Pool | A group of physical disks combined into a single managed storage resource |
| **Virtual Disk** | Virtual Disk / Storage Space | A logical disk created from a storage pool |
| **Storage Spaces** | Storage Spaces | Windows feature for creating resilient storage from multiple disks |
| **Quota** | Disk Quota | A limit on how much disk space a user or folder can use |
| **DFS** | Distributed File System | A service that makes multiple shares appear as a single namespace |
| **Shadow Copy** | Volume Shadow Copy | Automatic backups of files allowing previous version restoration |
| **iSCSI** | Internet Small Computer Systems Interface | A protocol for connecting to remote storage over a network |
| **RAID** | Redundant Array of Independent Disks | A method of combining disks for performance or redundancy |

---

## Overview
Windows Server's File and Storage Services role turns a server into a central file storage location for your organization. Users can access shared folders from any computer on the network, and administrators can control who has access to what.

Storage Spaces allows you to combine multiple physical disks into resilient storage pools — similar to RAID but managed through Windows.

---

## Prerequisites
- Windows Server with File and Storage Services role installed (usually installed by default)
- Administrator rights
- At least one additional disk for storage pools (optional)

---

## How to Open File and Storage Services

1. Open **Server Manager**
2. Click **File and Storage Services** in the left panel
3. Sub-sections: **Servers**, **Volumes**, **Disks**, **Storage Pools**, **Shares**, **iSCSI**

---

## 1. Creating a File Share

### GUI — Step by Step
1. Open **Server Manager** → **File and Storage Services** → **Shares**
2. In the right panel click **Tasks** → **New Share**
3. The **New Share Wizard** opens
4. Select share profile:
   - **SMB Share - Quick** — basic share, fastest setup
   - **SMB Share - Advanced** — includes quota and folder management
   - **NFS Share** — for Linux/Unix clients
5. Click **Next**
6. Select the server and volume → or type a custom path → **Next**
7. Enter a **Share name** e.g. `Documents` — the UNC path shows as `\\server\Documents`
8. Configure settings:
   - **Enable access-based enumeration** — users only see folders they have access to
   - **Allow caching of share** — for offline access
   - **Encrypt data access** — for security
9. Click **Next**
10. Configure **Permissions** — click **Customize permissions** to set NTFS and share permissions
11. Click **Next** → **Create** → **Close**

### PowerShell
```powershell
# Create a simple share
New-SmbShare -Name "Documents" `
  -Path "D:\Shares\Documents" `
  -Description "Company Documents" `
  -FullAccess "COMPANY\Domain Admins" `
  -ReadAccess "COMPANY\Domain Users"

# Create share with change access
New-SmbShare -Name "Projects" `
  -Path "D:\Shares\Projects" `
  -ChangeAccess "COMPANY\IT-Team" `
  -ReadAccess "COMPANY\Domain Users"

# View all shares
Get-SmbShare

# View share permissions
Get-SmbShareAccess -Name "Documents"
```

---

## 2. Understanding Share vs NTFS Permissions

This is one of the most important concepts in Windows file sharing. **Both** permission layers must be configured.

| | Share Permissions | NTFS Permissions |
|---|---|---|
| **Where set** | Share properties | File/folder properties → Security tab |
| **Applies** | Only over the network | Local AND network access |
| **Options** | Full Control, Change, Read | Full Control, Modify, Read & Execute, Read, Write, etc. |
| **Best Practice** | Grant **Everyone Full Control** at share level | Use NTFS permissions for granular control |

> **The effective permission is the MORE RESTRICTIVE of the two.**
> If Share = Read and NTFS = Full Control → user gets Read access

### Setting NTFS Permissions — GUI
1. Right-click the folder → **Properties**
2. Click the **Security** tab
3. Click **Edit** to modify permissions
4. Click **Add** to add a user or group
5. Select the user → check the permissions needed
6. Click **Apply** → **OK**

### PowerShell
```powershell
# Grant NTFS permission to a folder
$acl = Get-Acl "D:\Shares\Documents"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
  "COMPANY\IT-Team", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow"
)
$acl.SetAccessRule($rule)
Set-Acl "D:\Shares\Documents" $acl

# View current NTFS permissions
Get-Acl "D:\Shares\Documents" | Format-List
```

---

## 3. Managing Storage Pools

Storage Spaces lets you combine multiple physical disks into a resilient pool.

### GUI — Step by Step
1. Open **Server Manager** → **File and Storage Services** → **Storage Pools**
2. In the **Storage Pools** section click **Tasks** → **New Storage Pool**
3. Enter a **Name** e.g. `DataPool` → **Next**
4. Select the physical disks to include → **Next**
5. Review → **Create** → **Close**
6. Now create a **Virtual Disk** from the pool:
   - In **Virtual Disks** click **Tasks** → **New Virtual Disk**
   - Select the storage pool
   - Enter a name
   - Select layout:
     - **Simple** — no redundancy (like RAID 0)
     - **Mirror** — two-way or three-way mirror (like RAID 1)
     - **Parity** — space-efficient with fault tolerance (like RAID 5)
   - Set size
   - Click **Create**
7. Initialize and format the new disk in **Disk Management**

### PowerShell
```powershell
# View available disks for storage pools
Get-PhysicalDisk | Where-Object CanPool -eq $true

# Create a storage pool
New-StoragePool -FriendlyName "DataPool" `
  -StorageSubsystemFriendlyName "Windows Storage*" `
  -PhysicalDisks (Get-PhysicalDisk | Where-Object CanPool -eq $true)

# Create a mirrored virtual disk
New-VirtualDisk -StoragePoolFriendlyName "DataPool" `
  -FriendlyName "DataDisk" `
  -ResiliencySettingName "Mirror" `
  -Size 500GB `
  -ProvisioningType Fixed

# Initialize the new disk
Get-VirtualDisk -FriendlyName "DataDisk" | Get-Disk | Initialize-Disk -PartitionStyle GPT

# Create a volume
Get-VirtualDisk -FriendlyName "DataDisk" | Get-Disk | 
  New-Partition -AssignDriveLetter -UseMaximumSize |
  Format-Volume -FileSystem NTFS -NewFileSystemLabel "DataDisk"
```

---

## 4. Configuring Disk Quotas

Quotas limit how much disk space users can consume.

### GUI — Step by Step
1. Open **File Explorer** → right-click the volume → **Properties**
2. Click the **Quota** tab
3. Check **Enable quota management**
4. Check **Deny disk space to users exceeding quota limit**
5. Select **Limit disk space to** → enter limit e.g. `10 GB`
6. Set warning level e.g. `8 GB`
7. Click **Apply** → **OK**

For per-folder quotas using File Server Resource Manager (FSRM):

### PowerShell — FSRM Quotas
```powershell
# Install FSRM
Install-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools

# Create a quota on a folder
New-FsrmQuota -Path "D:\Shares\Documents" -Size 10GB

# Create a quota with soft limit (warning only)
New-FsrmQuota -Path "D:\Shares\HR" -Size 20GB -SoftLimit

# View all quotas
Get-FsrmQuota
```

---

## 5. Enabling Shadow Copies (Previous Versions)

Shadow Copies automatically save previous versions of files so users can restore accidentally deleted or modified files.

### GUI — Step by Step
1. Right-click the volume in **File Explorer** → **Properties**
2. Click the **Shadow Copies** tab
3. Select the volume → click **Enable**
4. Click **Settings** to configure:
   - **Storage area** — where shadow copies are saved
   - **Maximum size** — limit how much space shadow copies use
   - **Schedule** — how often copies are taken (default is twice daily)
5. Click **OK** → **OK**

### PowerShell
```powershell
# Enable shadow copies on C drive
Enable-ComputerRestore -Drive "D:\"

# Or using vssadmin
vssadmin add shadowstorage /for=D: /on=D: /maxsize=10GB

# Create a manual shadow copy
$class = [WMICLASS]"root\cimv2:Win32_ShadowCopy"
$class.Create("D:\", "ClientAccessible")

# List all shadow copies
vssadmin list shadows
```

---

## 6. Monitoring and Managing Shares

### GUI — Step by Step
1. Open **Server Manager** → **File and Storage Services** → **Shares**
2. Click any share to see:
   - Current connections
   - Open files
   - Share path
3. Right-click a share → **Properties** to modify settings
4. Right-click → **Remove Share** to delete

### PowerShell
```powershell
# View all shares
Get-SmbShare

# View open files on a share
Get-SmbOpenFile

# View active sessions
Get-SmbSession

# Disconnect all sessions from a share
Get-SmbSession | Close-SmbSession -Force

# Remove a share
Remove-SmbShare -Name "OldShare" -Force

# Modify share permissions
Grant-SmbShareAccess -Name "Documents" -AccountName "COMPANY\NewUser" -AccessRight Read -Force

# Revoke share access
Revoke-SmbShareAccess -Name "Documents" -AccountName "COMPANY\OldUser" -Force
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Can't access share from network | Firewall blocking SMB or share not created | Check Windows Firewall — allow File and Printer Sharing |
| Access denied to share | Permissions misconfigured | Check both Share AND NTFS permissions |
| Drive mapping disconnects | Unstable network or credential issues | Map drive with credentials, check Group Policy drive mapping |
| Shadow copies not working | VSS service stopped or disk full | Check Volume Shadow Copy service and available disk space |
| Storage pool degraded | A disk has failed | Check physical disk health and replace failed drive |
| Quota not enforced | Quota management not enabled | Enable quota management on the volume |

---

## Quick Reference

```powershell
# List all shares
Get-SmbShare

# Create a share
New-SmbShare -Name "ShareName" -Path "D:\Folder" -FullAccess "Domain Admins" -ReadAccess "Domain Users"

# Remove a share
Remove-SmbShare -Name "ShareName" -Force

# View share permissions
Get-SmbShareAccess -Name "ShareName"

# View open files
Get-SmbOpenFile

# View active sessions
Get-SmbSession

# List storage pools
Get-StoragePool

# List virtual disks
Get-VirtualDisk

# List physical disks
Get-PhysicalDisk
```

---

## Notes
- Best practice: Grant **Everyone - Full Control** at the Share level and use **NTFS permissions** for granular access control
- SMB port is **445** — ensure this is open on the firewall for file sharing to work
- Shadow Copies are NOT a replacement for backups — they are stored on the same volume and will be lost if the drive fails
- Storage Spaces mirror requires at least 2 disks — three-way mirror requires 3 disks
- Access-Based Enumeration hides folders users don't have access to — a security best practice
- DFS Namespace can make multiple shares appear as a single unified path — useful for large organizations

---

## Related Documents
- [Active Directory User Management](active-directory-user-management.md)
- [Active Directory Password & Group Policy](active-directory-password-local--group-policy.md)
- [Windows Event Viewer](windows-event-viewer.md)
- [Virtual Machine Backup and Restore](../Virtual-Machines/virtual-machine-backup-and-restore.md)
