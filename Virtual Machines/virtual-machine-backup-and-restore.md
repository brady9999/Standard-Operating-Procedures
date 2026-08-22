# Virtual Machine Backup and Restore
> A guide to backing up and restoring virtual machines across Proxmox, Hyper-V, and VMware environments.

**Category:** Virtual Machines  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Snapshot** | VM Snapshot | A point-in-time saved state of a VM — not a full backup |
| **Checkpoint** | Checkpoint | Hyper-V term for a snapshot |
| **Clone** | VM Clone | A full copy of a VM |
| **vzdump** | VZ Dump | Proxmox's built-in VM backup tool |
| **VBR** | Veeam Backup & Replication | Enterprise VM backup solution |
| **RTO** | Recovery Time Objective | How quickly you need to restore after a failure |
| **RPO** | Recovery Point Objective | How much data loss is acceptable — how old can the backup be |
| **Incremental** | Incremental Backup | Only backs up changes since the last backup |
| **Full Backup** | Full Backup | A complete backup of the entire VM |
| **Differential** | Differential Backup | Backs up changes since the last full backup |
| **CBS** | Changed Block Tracking | Tracks which disk blocks changed — enables efficient incremental backups |
| **Deduplication** | Deduplication | Eliminating duplicate data to save storage space |
| **Compression** | Compression | Reducing backup file size |
| **Replication** | VM Replication | Continuously copying a VM to another host for DR |
| **DR** | Disaster Recovery | The plan and process for recovering from a major failure |
| **3-2-1 Rule** | 3-2-1 Backup Rule | 3 copies, 2 different media, 1 offsite |

---

## Overview
VM backups are critical — a snapshot is NOT a backup. The 3-2-1 rule applies:
- **3** copies of your data
- **2** different storage media types (e.g. local disk + NAS)
- **1** copy offsite (cloud or remote location)

**Snapshot vs Backup:**

| | Snapshot | Backup |
|--|---------|--------|
| Speed to create | Very fast | Slower |
| Storage used | Grows over time | Fixed size |
| Independent of VM | ❌ — stored with VM | ✅ — separate file |
| Survives host failure | ❌ | ✅ |
| Use for | Short-term rollback | Real protection |

---

## 1. Proxmox VE Backup

### GUI — Create a Backup
1. Go to **Proxmox Web UI** → select the VM
2. Click **Backup** in the left menu
3. Click **Backup Now**
4. Configure:
   - **Storage**: Select backup storage target
   - **Mode**:
     - **Snapshot** — fastest, brief freeze
     - **Suspend** — pauses VM during backup
     - **Stop** — safest, shuts VM down during backup
   - **Compression**: zstd (recommended), gzip, lzo, or none
5. Click **Backup**

### GUI — Schedule Automated Backups
1. Go to **Datacenter** → **Backup**
2. Click **Add**
3. Configure:
   - **Node**: All or specific node
   - **Storage**: Where to store backups
   - **Schedule**: Daily, weekly, cron expression
   - **Selection**: All VMs, pool, or specific VMs
   - **Retention**: How many backups to keep
   - **Mode** and **Compression**
4. Click **Create**

### CLI — vzdump
```bash
# Backup a specific VM (VMID 100)
vzdump 100 --storage backup-storage --mode snapshot --compress zstd

# Backup all VMs
vzdump --all --storage backup-storage --mode snapshot --compress zstd

# Backup with retention (keep last 7 backups)
vzdump 100 --storage backup-storage --mode snapshot --compress zstd --prune-backups keep-last=7

# Backup to specific directory
vzdump 100 --dumpdir /mnt/backup/ --mode snapshot --compress zstd

# List backups
pvesm list backup-storage

# View backup logs
journalctl -u pve-backup

# Common vzdump options
# --mode snapshot|suspend|stop
# --compress  zstd|gzip|lzo|0 (none)
# --bwlimit   limit bandwidth in KiB/s
# --mailto    email address for notifications
```

### GUI — Restore a VM from Backup
1. Go to **Proxmox Web UI** → select the node → **local** or backup storage
2. Click **Backups**
3. Find the backup file → click **Restore**
4. Configure:
   - **VM ID**: New ID for the restored VM
   - **Storage**: Where to store the restored VM disks
   - **Start after restore**: Optional
5. Click **Restore**

### CLI — Restore
```bash
# Restore VM from backup file
qmrestore /var/lib/vz/dump/vzdump-qemu-100-2026_08_18-02_00_00.vma.zst 101

# Restore to specific storage
qmrestore /path/to/backup.vma.zst 101 --storage local-lvm

# Force restore (overwrite if VM ID exists)
qmrestore /path/to/backup.vma.zst 100 --force

# Restore and start immediately
qmrestore /path/to/backup.vma.zst 101 && qm start 101
```

---

## 2. Proxmox Snapshots

```bash
# Create a snapshot via GUI:
# VM → Snapshots → Take Snapshot
# Give it a name and optional description
# Check "Include RAM" to save memory state

# CLI — create snapshot
qm snapshot 100 "before-update" --description "Before applying patches" --vmstate 1

# List snapshots
qm listsnapshot 100

# Rollback to a snapshot
qm rollback 100 before-update

# Delete a snapshot
qm delsnapshot 100 before-update
```

> ⚠️ **Warning:** Snapshots grow over time as changes accumulate. Do not leave snapshots running for extended periods — they degrade performance and consume significant storage.

---

## 3. Hyper-V Backup

### PowerShell — Checkpoints (Snapshots)
```powershell
# Create a checkpoint
Checkpoint-VM -Name "MyVM" -SnapshotName "Before Windows Update"

# List all checkpoints for a VM
Get-VMCheckpoint -VMName "MyVM"

# Restore a checkpoint
Restore-VMCheckpoint -Name "Before Windows Update" -VMName "MyVM" -Confirm:$false

# Delete a checkpoint
Remove-VMCheckpoint -VMName "MyVM" -Name "Before Windows Update"

# Delete all checkpoints for a VM
Get-VMCheckpoint -VMName "MyVM" | Remove-VMCheckpoint
```

### PowerShell — VM Export (Full Backup)
```powershell
# Export a VM to a folder (VM must be stopped or use saved state)
Export-VM -Name "MyVM" -Path "D:\VMBackup"

# Export a running VM (takes longer)
Export-VM -Name "MyVM" -Path "D:\VMBackup" -AsJob

# Import a VM from export
Import-VM -Path "D:\VMBackup\MyVM\Virtual Machines\*.vmcx"

# Import with a new ID (copy — creates a new VM)
Import-VM -Path "D:\VMBackup\MyVM\Virtual Machines\*.vmcx" -Copy -GenerateNewId

# Compress backup after export
Compress-Archive -Path "D:\VMBackup\MyVM" -DestinationPath "D:\VMBackup\MyVM.zip"
```

### Windows Server Backup for Hyper-V
```powershell
# Install Windows Server Backup
Install-WindowsFeature Windows-Server-Backup

# Create a backup policy
$policy = New-WBPolicy
$backupLocation = New-WBBackupTarget -VolumePath D:
Add-WBBackupTarget -Policy $policy -Target $backupLocation
Set-WBSchedule -Policy $policy -Schedule 02:00
Set-WBPolicy -Policy $policy

# Run backup immediately
Start-WBBackup -Policy $policy

# View backup status
Get-WBJob -Previous 5
```

---

## 4. VMware ESXi Backup

### PowerCLI — Snapshots
```powershell
# Connect to ESXi
Connect-VIServer -Server esxi.company.com

# Create a snapshot
New-Snapshot -VM "MyVM" -Name "Before Update" -Description "Pre-patch snapshot" -Memory -Quiesce

# List snapshots
Get-Snapshot -VM "MyVM"

# Revert to snapshot
Set-VM -VM "MyVM" -SnapShot (Get-Snapshot -VM "MyVM" -Name "Before Update") -Confirm:$false

# Remove snapshot
Remove-Snapshot -Snapshot (Get-Snapshot -VM "MyVM" -Name "Before Update") -Confirm:$false
```

### PowerCLI — VM Clone
```powershell
# Clone a VM (full clone — independent copy)
New-VM -Name "MyVM-Clone" -VM "MyVM" -ResourcePool (Get-ResourcePool) -Datastore (Get-Datastore "datastore1")

# Clone from snapshot (linked clone — shares base disk)
New-VM -Name "MyVM-LinkedClone" -VM "MyVM" -LinkedClone -ReferenceSnapshot (Get-Snapshot -VM "MyVM" -Name "Base")
```

---

## 5. Cloud/Offsite Backup with Backblaze B2

Backblaze B2 provides cheap offsite object storage — excellent for VM backup archives.

```bash
# Install rclone (Linux)
curl https://rclone.org/install.sh | sudo bash

# Configure rclone for B2
rclone config
# Follow prompts: new remote → b2 → enter account ID and application key

# Sync local backups to B2
rclone sync /var/lib/vz/dump/ b2:my-backup-bucket/proxmox/ --progress

# Copy specific backup to B2
rclone copy /var/lib/vz/dump/vzdump-qemu-100-2026_08_18.vma.zst b2:my-backup-bucket/proxmox/

# List files in B2
rclone ls b2:my-backup-bucket/

# Restore from B2
rclone copy b2:my-backup-bucket/proxmox/vzdump-qemu-100-2026_08_18.vma.zst /var/lib/vz/dump/

# Automate with cron — run after nightly Proxmox backup
# Add to crontab: 0 4 * * * rclone sync /var/lib/vz/dump/ b2:my-backup-bucket/proxmox/ --min-age 1h
```

---

## 6. Automated Backup Script (Proxmox + B2)

```bash
#!/bin/bash
# /opt/scripts/vm-backup.sh
# Backs up all VMs and syncs to Backblaze B2

BACKUP_STORAGE="local"
BACKUP_DIR="/var/lib/vz/dump"
B2_REMOTE="b2:my-backup-bucket/proxmox"
LOG="/var/log/vm-backup.log"
KEEP_LOCAL=7
KEEP_REMOTE=30

echo "$(date) — Starting VM backup" >> $LOG

# Backup all VMs
vzdump --all \
  --storage $BACKUP_STORAGE \
  --mode snapshot \
  --compress zstd \
  --prune-backups keep-last=$KEEP_LOCAL \
  --quiet >> $LOG 2>&1

# Sync to B2
rclone sync $BACKUP_DIR $B2_REMOTE \
  --min-age 1h \
  --log-file $LOG \
  --log-level INFO

# Remove old remote backups (older than 30 days)
rclone delete $B2_REMOTE --min-age ${KEEP_REMOTE}d >> $LOG 2>&1

echo "$(date) — Backup complete" >> $LOG
```

```bash
# Make executable
chmod +x /opt/scripts/vm-backup.sh

# Schedule in cron (run at 2 AM daily)
crontab -e
# Add: 0 2 * * * /opt/scripts/vm-backup.sh
```

---

## 7. Backup Verification

A backup that has never been tested is not a backup.

```bash
# Proxmox — test restore to a different VM ID
qmrestore /var/lib/vz/dump/vzdump-qemu-100-2026_08_18.vma.zst 999
qm start 999
# Verify the VM boots and data is intact
qm stop 999
qm destroy 999

# Check backup file integrity
zstd --test /var/lib/vz/dump/vzdump-qemu-100-2026_08_18.vma.zst

# Verify B2 checksums
rclone check /var/lib/vz/dump/ b2:my-backup-bucket/proxmox/ --one-way
```

---

## Backup Strategy Recommendations

```
Daily backups (automated):
- Full VM backup of all production VMs
- Retain 7 days locally
- Sync to offsite storage (B2 or NAS)
- Retain 30 days offsite

Before major changes:
- Take a snapshot immediately before
- Delete snapshot after confirming success

Monthly:
- Test restore of at least one VM
- Verify offsite backup integrity
- Review retention policies

Documentation:
- Document what's backed up and where
- Document restore procedures
- Store recovery key information securely
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Backup fails — not enough space | Backup storage full | Prune old backups, add storage |
| Snapshot grows too large | Left running too long | Delete snapshot, consolidate disks |
| Restore fails — VM ID conflict | VM ID already exists | Use `--force` or choose a new ID |
| VM won't start after restore | Storage path changed | Update VM config storage references |
| Slow backup | No compression or large VM | Enable zstd compression, schedule during off-hours |
| B2 upload fails | Network or credential issue | Test rclone config: `rclone lsd b2:bucket` |

---

## Quick Reference

```bash
# Proxmox backup
vzdump 100 --storage local --mode snapshot --compress zstd

# Proxmox restore
qmrestore /path/to/backup.vma.zst 101

# Proxmox snapshot
qm snapshot 100 "name"
qm rollback 100 "name"
qm listsnapshot 100

# Hyper-V checkpoint
Checkpoint-VM -Name "VM" -SnapshotName "name"
Restore-VMCheckpoint -Name "name" -VMName "VM"

# Hyper-V export/import
Export-VM -Name "VM" -Path "D:\Backup"
Import-VM -Path "D:\Backup\VM\*.vmcx"

# Sync to B2
rclone sync /var/lib/vz/dump/ b2:bucket/proxmox/
```

---

## Notes
- Snapshots are NOT backups — they protect against software mistakes but not hardware failure
- Always test your backups by actually restoring them — discover problems before a crisis
- The 3-2-1 rule is the minimum standard — 3 copies, 2 media types, 1 offsite
- vzdump with `--mode snapshot` causes minimal VM disruption — use this for production VMs
- Backblaze B2 is approximately $6/TB/month — extremely cost-effective for offsite VM archives
- Schedule backups during off-peak hours — they consume significant I/O

---

## Related Documents
- [Virtual Machine Security Basics](virtual-machine-security-basics.md)
- [Virtual Networks Basics](virtual-networks-basics.md)
- [Hyper-V Installation](../Windows/hyperv-installation.md)
