# Virtual Machine Security Basics
> A guide to securing virtual machines, hypervisors, and virtualization environments.

**Category:** Virtual Machines  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **VM Escape** | VM Escape | An attack where a process inside a VM breaks out to the hypervisor |
| **Hypervisor Security** | Hypervisor Security | Protecting the software layer that manages VMs |
| **Isolation** | VM Isolation | Ensuring VMs cannot access each other's resources |
| **Hardening** | Hardening | Reducing the attack surface of a system |
| **Template** | VM Template | A pre-configured VM image used as a starting point |
| **Golden Image** | Golden Image | A hardened, approved VM template used for deployments |
| **RBAC** | Role-Based Access Control | Controlling who can manage which VMs |
| **Audit Log** | Audit Log | A record of administrative actions on the hypervisor |
| **Secure Boot** | Secure Boot | UEFI feature ensuring only trusted code runs at boot |
| **vTPM** | Virtual Trusted Platform Module | A virtual TPM chip enabling BitLocker and other security features in VMs |
| **Network Segmentation** | Network Segmentation | Isolating VMs on separate networks to limit lateral movement |
| **Least Privilege** | Least Privilege | Giving VMs and admins only the access they need |
| **Patch Management** | Patch Management | Keeping hypervisor and VM software up to date |
| **IDS/IPS** | Intrusion Detection/Prevention | Monitoring for and blocking malicious activity |

---

## Overview
Virtualization introduces unique security challenges — a single compromised hypervisor could expose all VMs running on it. Security must be applied at multiple layers:

```
Layer 1 — Physical Host Security
Layer 2 — Hypervisor Security
Layer 3 — VM Network Security
Layer 4 — Guest OS Security (inside each VM)
Layer 5 — Application Security (inside each VM)
```

---

## 1. Hypervisor Security

### Proxmox VE Hardening

```bash
# 1. Keep Proxmox updated
apt update && apt full-upgrade -y

# 2. Secure the Proxmox web interface
# Change default HTTPS port (optional)
# Restrict access by IP in /etc/hosts.allow

# 3. Restrict web UI access to management network only
# Add to /etc/hosts.allow:
echo "pveproxy: 192.168.99.0/24" >> /etc/hosts.allow
echo "pveproxy: ALL" >> /etc/hosts.deny

# 4. Use SSH keys only — disable password auth
nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
# Set: PermitRootLogin prohibit-password
systemctl restart ssh

# 5. Enable firewall on Proxmox node
# Datacenter → Firewall → Enable
# Add rules to allow only necessary traffic

# 6. Restrict root access — create non-root admin
# Datacenter → Permissions → Users → Add user
# Assign appropriate roles

# 7. Enable two-factor authentication
# Datacenter → Permissions → Two Factor → Add TOTP

# 8. Review and disable unused services
systemctl list-units --type=service --state=running
systemctl disable --now servicename
```

### Proxmox Firewall Configuration

```bash
# Enable datacenter firewall
# Datacenter → Firewall → Options → Firewall: Yes

# Enable node-level firewall
# Node → Firewall → Options → Firewall: Yes

# Enable VM-level firewall
# VM → Firewall → Options → Firewall: Yes

# Add rules via CLI
pvesh create /nodes/proxmox/firewall/rules \
  --action ACCEPT \
  --type in \
  --proto tcp \
  --dport 8006 \
  --source 192.168.99.0/24 \
  --comment "Allow web UI from management VLAN"
```

### Hyper-V Security

```powershell
# Enable Hyper-V host firewall rules only for needed ports
Get-NetFirewallRule | Where-Object {$_.DisplayName -like "*Hyper-V*"} |
  Select-Object DisplayName, Enabled, Action

# Disable unnecessary Hyper-V features
Disable-WindowsOptionalFeature -Online -FeatureName "Microsoft-Hyper-V-Tools-All" -NoRestart

# Enable Credential Guard (protects against credential theft)
# Group Policy: Computer Config → Admin Templates → System → Device Guard → Turn on Virtualization Based Security

# Enable Secure Boot for VMs
Set-VMFirmware -VMName "MyVM" -EnableSecureBoot On

# Enable vTPM for VMs (required for BitLocker)
Add-VMTPMState -VMName "MyVM"
```

---

## 2. VM Network Isolation

Proper network segmentation prevents a compromised VM from reaching other VMs or the hypervisor.

### Network Isolation Strategy

```
Recommended VLAN structure for virtualized environments:

VLAN 10 — Management (Proxmox web UI, SSH to host)
  → Only accessible from admin workstations
  → No VMs on this VLAN except jump hosts

VLAN 20 — Production VMs
  → Application servers
  → Limited inbound access from internet

VLAN 30 — Development/Test VMs
  → No access to production VLAN
  → Internet access for package downloads

VLAN 40 — Database VMs
  → Only accessible from VLAN 20 (production)
  → No direct internet access

VLAN 99 — DMZ (internet-facing VMs)
  → Strict outbound filtering
  → No access to internal VLANs
```

### Proxmox — VM Network Isolation

```bash
# Create separate bridges for each security zone
auto vmbr10
iface vmbr10 inet static
    address 10.10.0.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    # Management network — Proxmox host only

auto vmbr20
iface vmbr20 inet static
    address 10.20.0.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    # Production network

auto vmbr40
iface vmbr40 inet static
    address 10.40.0.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    # Database network — isolated
```

```bash
# Assign VMs to appropriate bridges
qm set 100 -net0 virtio,bridge=vmbr20    # Production VM
qm set 200 -net0 virtio,bridge=vmbr40    # Database VM

# Database VM should have NO internet — use iptables on host to enforce
iptables -A FORWARD -i vmbr40 -o ens18 -j DROP
iptables -A FORWARD -i vmbr40 -o vmbr10 -j DROP
```

### Hyper-V — VM Network Isolation

```powershell
# Create dedicated internal switch for database VMs
New-VMSwitch -Name "DB-Internal" -SwitchType Internal

# Assign database VM to isolated switch
Connect-VMNetworkAdapter -VMName "DB-Server" -SwitchName "DB-Internal"

# Block internet access using Windows Firewall on the VM
# Or use NSGs in Azure/Hyper-V environment

# Verify VM network assignments
Get-VMNetworkAdapter -VMName "DB-Server" | Select-Object VMName, SwitchName
```

---

## 3. Guest OS Security (Inside VMs)

Every VM should be hardened as if it were a physical server.

### Linux VM Hardening

```bash
# 1. Update the system immediately after deployment
apt update && apt full-upgrade -y

# 2. Enable UFW firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable

# 3. Disable root login via SSH
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart ssh

# 4. Use SSH keys only
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
systemctl restart ssh

# 5. Install fail2ban
apt install fail2ban -y
systemctl enable --now fail2ban

# 6. Remove unnecessary packages
apt remove --purge telnet ftp rsh-client -y

# 7. Disable unnecessary services
systemctl disable --now cups avahi-daemon

# 8. Set up automatic security updates
apt install unattended-upgrades -y
dpkg-reconfigure --priority=low unattended-upgrades

# 9. Enable audit logging
apt install auditd -y
systemctl enable --now auditd

# 10. Configure strong password policy in /etc/security/pwquality.conf
echo "minlen = 12" >> /etc/security/pwquality.conf
echo "minclass = 3" >> /etc/security/pwquality.conf
```

### Windows VM Hardening

```powershell
# 1. Enable Windows Firewall
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# 2. Enable Secure Boot (set in Hyper-V VM firmware settings)
Set-VMFirmware -VMName "MyVM" -EnableSecureBoot On

# 3. Enable BitLocker (requires vTPM)
Enable-BitLocker -MountPoint "C:" -EncryptionMethod Aes256 -RecoveryPasswordProtector

# 4. Disable SMBv1
Set-SmbServerConfiguration -EnableSMB1Protocol $false

# 5. Enable Windows Defender
Set-MpPreference -DisableRealtimeMonitoring $false

# 6. Configure Windows Update for automatic security updates
# Via Group Policy or:
$updateSettings = (New-Object -com "Microsoft.Update.AutoUpdate").Settings
$updateSettings.NotificationLevel = 4  # Auto install
$updateSettings.Save()

# 7. Disable unnecessary services
Stop-Service -Name "Spooler" -Force    # If no printing needed
Set-Service -Name "Spooler" -StartupType Disabled

# 8. Enable audit policy
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable

# 9. Set account lockout policy
net accounts /lockoutthreshold:5 /lockoutduration:30 /lockoutwindow:30

# 10. Remove unnecessary Windows features
Disable-WindowsOptionalFeature -Online -FeatureName "TelnetClient"
```

---

## 4. Access Control for Hypervisor Admins

### Proxmox RBAC

```bash
# View available roles
pveum role list

# Create a custom role with limited permissions
pveum role add VM-Operator --privs "VM.PowerMgmt VM.Console VM.Monitor"

# Create a user
pveum user add jsmith@pve --password "TempPass123!" --email "jsmith@company.com"

# Assign user a role on specific VMs only
pveum acl modify /vms/100 --users jsmith@pve --roles VM-Operator

# Assign user to a resource pool
pveum pool modify production --vms 100,101,102
pveum acl modify /pool/production --users jsmith@pve --roles VM-Operator

# View user permissions
pveum acl list

# Remove user access
pveum acl delete /vms/100 --users jsmith@pve
```

### Hyper-V RBAC

```powershell
# Grant user Hyper-V Administrator rights
Add-LocalGroupMember -Group "Hyper-V Administrators" -Member "DOMAIN\jsmith"

# Or use Authorization Manager for fine-grained control
# Hyper-V Manager → Action → Hyper-V Settings → Authorization Manager

# View current Hyper-V admin group members
Get-LocalGroupMember -Group "Hyper-V Administrators"
```

---

## 5. VM Template Security

Creating secure golden images ensures all deployed VMs start from a hardened baseline.

```bash
# Steps to create a hardened Proxmox template:

# 1. Create a new VM with minimal installation
# 2. Apply all OS updates
apt update && apt full-upgrade -y

# 3. Apply all hardening steps from section 3

# 4. Install common monitoring/management tools
apt install -y htop curl wget vim net-tools

# 5. Clear machine-specific data
# Clear SSH host keys (regenerated on first boot)
rm -f /etc/ssh/ssh_host_*

# Clear machine ID (regenerated on first boot)
truncate -s 0 /etc/machine-id
rm /var/lib/dbus/machine-id
ln -s /etc/machine-id /var/lib/dbus/machine-id

# Clear logs
find /var/log -type f -exec truncate -s 0 {} \;

# Clear bash history
history -c
cat /dev/null > ~/.bash_history

# 6. Shut down the VM
shutdown -h now

# 7. Convert to template in Proxmox
# Right-click VM → Convert to template
# Or via CLI:
qm template 200
```

---

## 6. Monitoring and Detecting VM Security Issues

```bash
# Monitor VM resource usage (detect cryptomining, unusual activity)
# On Proxmox host:
qm monitor 100
# In the monitor: info cpus, info mem, info block

# Check for unusual network traffic from VMs
tcpdump -i vmbr0 -n port not 22 and port not 443

# Monitor VM console for suspicious activity
# Proxmox: VM → Console

# Check VM audit logs in Proxmox
journalctl | grep "VM 100"

# Monitor disk I/O (detect ransomware activity)
iostat -x 5

# Check for VMs with unusual CPU usage
qm list | while read vmid rest; do
  cpu=$(qm status $vmid 2>/dev/null | grep cpu | awk '{print $2}')
  echo "VM $vmid: CPU $cpu"
done
```

### Proxmox Audit Logging

```bash
# View Proxmox task history
pvesh get /cluster/tasks

# View recent API calls
journalctl -u pveproxy -f

# View authentication logs
journalctl | grep "pve-auth"
```

---

## 7. Incident Response for VMs

```bash
# Isolate a compromised VM immediately
# Remove it from all networks:
qm set 100 -net0 none    # Proxmox — remove network adapter

# Or via Hyper-V PowerShell:
Disconnect-VMNetworkAdapter -VMName "CompromisedVM"

# Take a forensic snapshot BEFORE shutting down
# Memory state may contain evidence
qm snapshot 100 "forensic-$(date +%Y%m%d-%H%M%S)" --vmstate 1

# Then suspend the VM (preserve memory state)
qm suspend 100

# Document everything:
# - When was the compromise detected?
# - What indicators of compromise (IOCs) were found?
# - Which VMs and networks were affected?
# - What actions were taken and when?

# Restore from known-good backup if needed
qmrestore /path/to/last-known-good-backup.vma.zst 200
```

---

## Security Checklist

```
Hypervisor:
✅ Hypervisor software kept updated
✅ Management interface access restricted to management VLAN
✅ Two-factor authentication on hypervisor admin accounts
✅ RBAC configured — least privilege for all admins
✅ Audit logging enabled
✅ SSH hardened — keys only, no root login
✅ Unused services disabled

VM Networks:
✅ VMs separated into security zones by VLAN
✅ Database VMs have no direct internet access
✅ Management VLAN separated from production VMs
✅ Firewall rules between VLANs enforced

Guest OS:
✅ OS fully updated after deployment
✅ Host firewall enabled
✅ Unnecessary services disabled
✅ SSH hardened (Linux) or RDP restricted (Windows)
✅ fail2ban or equivalent installed
✅ Audit logging enabled

Backup and Recovery:
✅ Regular automated backups configured
✅ Backups tested by performing a restore
✅ Offsite copy maintained
✅ Backup access restricted

Monitoring:
✅ Unusual CPU/network activity monitoring configured
✅ Hypervisor logs reviewed regularly
✅ Failed authentication attempts monitored
```

---

## Notes
- Hypervisor security is paramount — a compromised hypervisor exposes ALL VMs
- Network isolation is your most important VM security control — segment aggressively
- Never run development/test VMs on the same network as production
- Golden images ensure consistent security baseline across all deployments
- Regular patching of both hypervisor and guest OS is non-negotiable
- Test your incident response procedures before you need them

---

## Related Documents
- [Virtual Machine Routing](virtual-machine-routing.md)
- [Virtual Machine Backup and Restore](virtual-machine-backup-and-restore.md)
- [Linux Security Basics](../Linux/linux-security-basics.md)
- [Network Security Basics](../Security/network-security-basics.md)
