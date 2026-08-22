# Linux Security Basics
> A guide to fundamental Linux security practices, hardening techniques, and monitoring tools.

**Category:** Linux  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **SSH** | Secure Shell | Encrypted protocol for remote access to Linux systems |
| **sudo** | Superuser Do | Run commands with elevated privileges without logging in as root |
| **UFW** | Uncomplicated Firewall | A user-friendly front-end for iptables firewall management |
| **iptables** | iptables | The underlying Linux firewall framework |
| **fail2ban** | fail2ban | A tool that bans IPs after too many failed login attempts |
| **SELinux** | Security-Enhanced Linux | A mandatory access control system built into Linux |
| **AppArmor** | AppArmor | A Linux security module that restricts application capabilities |
| **PAM** | Pluggable Authentication Modules | A framework for authentication policies in Linux |
| **Sudoers** | Sudoers File | The file defining who can use sudo and what they can run |
| **chroot** | Change Root | Isolating a process to a specific directory subtree |
| **SUID** | Set User ID | A permission bit allowing a file to run as the owner, not the executor |
| **Audit** | Linux Audit | A framework for logging security-relevant events |
| **CIS** | Center for Internet Security | An organization that publishes security benchmarks and best practices |
| **CVE** | Common Vulnerabilities and Exposures | A public database of known security vulnerabilities |

---

## Overview
Linux security requires a layered approach — no single control is sufficient. The key principles are:
- **Least Privilege** — users and services get only the access they need
- **Defense in Depth** — multiple overlapping security controls
- **Regular Patching** — keeping software up to date
- **Monitoring** — knowing what's happening on your system
- **Minimal Attack Surface** — disable and remove what you don't need

---

## 1. User and Access Security

### SSH Key Authentication (Disable Password Login)

```bash
# Generate SSH key pair on your LOCAL machine
ssh-keygen -t ed25519 -C "brady@company.com"
# This creates:
# ~/.ssh/id_ed25519      (private key — keep this secret)
# ~/.ssh/id_ed25519.pub  (public key — goes on the server)

# Copy your public key to the server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server-ip

# Or manually add to authorized_keys on the server
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh/```

### Hardening SSH Configuration

```bash
sudo nano /etc/ssh/sshd_config
```

```ini
# Key SSH hardening settings

# Change default port (reduces automated scanning)
Port 2222

# Disable root login — always use sudo
PermitRootLogin no

# Disable password authentication — use keys only
PasswordAuthentication no
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Limit which users can SSH in
AllowUsers brady admin

# Disable unused authentication methods
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no

# Connection limits
MaxAuthTries 3
MaxSessions 5
LoginGraceTime 30

# Idle timeout (disconnect after 10 minutes idle)
ClientAliveInterval 600
ClientAliveCountMax 0

# Disable X11 forwarding if not needed
X11Forwarding no

# Disable TCP forwarding if not needed
AllowTcpForwarding no

# Use strong ciphers only
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

```bash
# Test config before restarting
sudo sshd -t

# Restart SSH (WARNING — keep current session open in case of errors)
sudo systemctl restart ssh

# Verify SSH is running
sudo systemctl status ssh
```

---

## 2. Sudo Configuration

```bash
# Edit sudoers file (always use visudo — validates syntax before saving)
sudo visudo

# View current sudoers
sudo cat /etc/sudoers

# Check sudoers.d directory for additional rules
ls /etc/sudoers.d/
```

### Common Sudoers Configurations

```bash
# Allow user to run all commands with sudo
brady ALL=(ALL:ALL) ALL

# Allow user to run all commands without password (convenient but less secure)
brady ALL=(ALL:ALL) NOPASSWD: ALL

# Allow user to run only specific commands
brady ALL=(ALL) /bin/systemctl restart nginx, /usr/bin/apt update

# Allow a group to use sudo
%sudo ALL=(ALL:ALL) ALL
%IT-Admin ALL=(ALL:ALL) ALL

# Allow user to run commands as a specific user
brady ALL=(webuser) /usr/bin/python3

# Restrict sudo to specific hosts
brady server1=(ALL:ALL) ALL
```

### Sudo Best Practices
```bash
# View what sudo commands a user can run
sudo -l

# Log all sudo commands (enabled by default in most distros)
# Logs appear in /var/log/auth.log or journalctl

# Check sudo log
sudo grep sudo /var/log/auth.log | tail -20
journalctl | grep sudo | tail -20
```

---

## 3. Firewall Configuration (UFW)

```bash
# Install UFW
sudo apt install ufw -y

# Check status
sudo ufw status
sudo ufw status verbose

# Set default policies (deny all incoming, allow all outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (do this BEFORE enabling UFW to avoid locking yourself out)
sudo ufw allow ssh
# Or if using custom port:
sudo ufw allow 2222/tcp

# Allow common services
sudo ufw allow http          # Port 80
sudo ufw allow https         # Port 443
sudo ufw allow 5432/tcp      # PostgreSQL
sudo ufw allow 8000/tcp      # Gunicorn/development

# Allow from specific IP only
sudo ufw allow from 192.168.1.0/24 to any port 22
sudo ufw allow from 192.168.99.10 to any port 5432

# Enable UFW
sudo ufw enable

# Disable a rule
sudo ufw delete allow http
sudo ufw delete allow 8000/tcp

# Reset UFW (removes all rules)
sudo ufw reset

# View numbered rules (easier to delete specific rules)
sudo ufw status numbered
sudo ufw delete 3      # Delete rule number 3
```

---

## 4. fail2ban — Brute Force Protection

fail2ban monitors log files and bans IPs that show malicious signs such as too many password failures.

```bash
# Install fail2ban
sudo apt install fail2ban -y

# Start and enable
sudo systemctl enable --now fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

### Configuring fail2ban

```bash
# Never edit jail.conf directly — create a local override
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

```ini
# /etc/fail2ban/jail.local — key settings

[DEFAULT]
# Ban time in seconds (3600 = 1 hour)
bantime = 3600

# Time window for counting failures (600 = 10 minutes)
findtime = 600

# Number of failures before ban
maxretry = 5

# Email notifications (optional)
destemail = brady@company.com
sendername = fail2ban
action = %(action_mwl)s

[sshd]
enabled = true
port = 2222    # Match your SSH port
logpath = %(sshd_log)s
maxretry = 3
bantime = 86400    # 24 hour ban for SSH failures
```

```bash
# Restart fail2ban after config changes
sudo systemctl restart fail2ban

# View banned IPs
sudo fail2ban-client status sshd

# Manually ban an IP
sudo fail2ban-client set sshd banip 192.168.1.100

# Unban an IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# View fail2ban logs
sudo tail -f /var/log/fail2ban.log
journalctl -u fail2ban -f
```

---

## 5. System Updates and Patch Management

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y

# Full upgrade including dependency changes
sudo apt full-upgrade -y

# Remove unused packages
sudo apt autoremove -y

# Check for security-specific updates
sudo apt list --upgradable 2>/dev/null | grep security

# Install automatic security updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades

# Configure automatic updates
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

```bash
# Check which packages need security updates
sudo apt-get -s upgrade | grep -i security

# View recently installed packages
grep " install " /var/log/dpkg.log | tail -20

# Check package version
dpkg -l packagename
apt show packagename
```

---

## 6. File Permission Auditing

```bash
# Find world-writable files (anyone can write — security risk)
sudo find / -type f -perm -o+w -not -path "/proc/*" 2>/dev/null

# Find SUID files (run as owner — audit these)
sudo find / -type f -perm -4000 2>/dev/null

# Find SGID files (run as group)
sudo find / -type f -perm -2000 2>/dev/null

# Find files with no owner
sudo find / -nouser -o -nogroup 2>/dev/null

# Find world-writable directories
sudo find / -type d -perm -o+w 2>/dev/null

# Check important file permissions
ls -la /etc/passwd          # Should be 644
ls -la /etc/shadow          # Should be 640 or 000
ls -la /etc/sudoers         # Should be 440
ls -la /etc/ssh/sshd_config # Should be 600

# Fix insecure permissions
sudo chmod 644 /etc/passwd
sudo chmod 640 /etc/shadow
sudo chmod 440 /etc/sudoers
```

---

## 7. Log Monitoring

```bash
# System logs
journalctl -f                           # Follow all system logs
journalctl -p err -f                    # Follow errors only
journalctl --since today                # Today's logs

# Authentication logs
sudo tail -f /var/log/auth.log          # Login attempts, sudo use
journalctl _SYSTEMD_UNIT=ssh.service    # SSH specific logs

# Failed login attempts
sudo grep "Failed password" /var/log/auth.log | tail -20
sudo grep "Invalid user" /var/log/auth.log | tail -20

# Successful logins
sudo grep "Accepted" /var/log/auth.log | tail -20

# System messages
sudo tail -f /var/log/syslog
sudo dmesg -T | tail -20

# Application logs
sudo tail -f /var/log/nginx/error.log
journalctl -u myapp.service -f

# Audit log (if auditd installed)
sudo ausearch -m USER_LOGIN -ts today
sudo ausearch -m EXECVE -ts today | head -50
```

### Setting Up Audit Logging

```bash
# Install auditd
sudo apt install auditd -y
sudo systemctl enable --now auditd

# Add audit rules
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
sudo auditctl -w /etc/sudoers -p wa -k sudoers_changes
sudo auditctl -w /etc/ssh/sshd_config -p wa -k ssh_config
sudo auditctl -a always,exit -F arch=b64 -S execve -k command_execution

# Make rules persistent
sudo nano /etc/audit/rules.d/audit.rules
```

```
# /etc/audit/rules.d/audit.rules
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
-w /etc/sudoers -p wa -k sudoers_changes
-w /etc/ssh/sshd_config -p wa -k ssh_config_changes
-w /var/log/auth.log -p wa -k auth_log
-a always,exit -F arch=b64 -S execve -k command_execution
```

---

## 8. Security Hardening Checklist

### User Security
```bash
# Lock unused accounts
sudo passwd -l username         # Lock account
sudo passwd -u username         # Unlock account

# Set password expiry
sudo chage -M 90 username       # Password expires in 90 days
sudo chage -l username          # View password aging info

# Check for accounts with empty passwords
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# Check for accounts with UID 0 (root-equivalent)
awk -F: '($3 == 0) {print $1}' /etc/passwd
# Only "root" should appear here
```

### Network Hardening
```bash
# Disable IPv6 if not used
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Enable SYN flood protection
echo "net.ipv4.tcp_syncookies = 1" | sudo tee -a /etc/sysctl.conf

# Disable IP forwarding (unless this is a router)
echo "net.ipv4.ip_forward = 0" | sudo tee -a /etc/sysctl.conf

# Disable ICMP redirects
echo "net.ipv4.conf.all.accept_redirects = 0" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.conf.all.send_redirects = 0" | sudo tee -a /etc/sysctl.conf

# Apply all sysctl changes
sudo sysctl -p
```

### Service Hardening
```bash
# List all running services — disable what you don't need
sudo systemctl list-units --type=service --state=running

# Disable and stop unnecessary services
sudo systemctl disable --now servicename

# Common services to consider disabling on servers:
# bluetooth, cups (printing), avahi-daemon, rpcbind (if no NFS)

# Check for services listening on unexpected ports
sudo ss -tuln
```

---

## 9. Intrusion Detection — Basic Checks

```bash
# Check for recently modified system files
sudo find /etc /bin /sbin /usr -newer /var/log/dpkg.log -type f 2>/dev/null

# Check running processes for unexpected ones
ps aux | grep -v "^root\|^www-data\|^postgres\|^brady" | grep -v "grep"

# Check network connections — look for unexpected outbound connections
ss -tupn
netstat -tupn

# Check crontabs for unexpected entries
crontab -l                          # Current user
sudo crontab -l                     # Root
ls -la /etc/cron*                   # System crontabs
cat /etc/crontab

# Check startup scripts
ls -la /etc/init.d/
ls -la /etc/systemd/system/

# Check for unusual SUID/SGID files
sudo find / -perm -4000 -o -perm -2000 2>/dev/null | sort

# Check /tmp for suspicious files
ls -lat /tmp/

# Check last logins
last | head -20
lastb | head -20    # Failed logins
```

---

## 10. Quick Security Audit Script

```bash
#!/bin/bash
# Basic security check script

echo "=== System Security Audit ==="
echo ""

echo "--- Failed Login Attempts (last 20) ---"
grep "Failed password" /var/log/auth.log 2>/dev/null | tail -20

echo ""
echo "--- Recent Successful Logins ---"
last | head -10

echo ""
echo "--- Listening Ports ---"
ss -tuln

echo ""
echo "--- UFW Status ---"
ufw status

echo ""
echo "--- fail2ban Status ---"
fail2ban-client status 2>/dev/null

echo ""
echo "--- Users with sudo access ---"
grep -Po '^sudo.+:\K.*$' /etc/group | tr ',' '\n'

echo ""
echo "--- SUID Files ---"
find / -perm -4000 -type f 2>/dev/null | sort

echo ""
echo "--- Pending Security Updates ---"
apt list --upgradable 2>/dev/null | grep -i security
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Locked out via SSH | Wrong SSH config or key | Access via console, fix sshd_config |
| fail2ban banning legitimate IPs | Low maxretry threshold | Whitelist IPs: `ignoreip` in jail.local |
| UFW blocking legitimate traffic | Missing allow rule | `ufw allow port/tcp` |
| SSH slow to connect | DNS lookup timeout | Add `UseDNS no` to sshd_config |
| Sudo not working | User not in sudo group | `usermod -aG sudo username` |
| Permission denied errors | Wrong file permissions | Use `chmod` and `chown` to fix |

---

## Quick Reference

```bash
# SSH hardening
sudo nano /etc/ssh/sshd_config       # Edit config
sudo sshd -t                          # Test config
sudo systemctl restart ssh            # Apply changes

# UFW
sudo ufw status                       # Check status
sudo ufw allow 22                     # Allow SSH
sudo ufw enable                       # Enable firewall

# fail2ban
sudo fail2ban-client status sshd      # Check SSH jail
sudo fail2ban-client set sshd unbanip IP  # Unban IP

# Updates
sudo apt update && sudo apt upgrade   # Update system

# Logs
journalctl -f                         # Follow all logs
sudo grep "Failed" /var/log/auth.log  # Failed logins

# Permissions
sudo find / -perm -4000 2>/dev/null   # Find SUID files
ls -la /etc/passwd /etc/shadow        # Check critical file perms
```

---

## Notes
- Always test SSH config with `sshd -t` before restarting — a bad config can lock you out
- Never disable UFW's SSH rule before ensuring you have console access as a fallback
- Keep at least one password-authenticated backup access method until SSH keys are confirmed working
- fail2ban is one of the most effective tools against automated attacks — install it on every internet-facing server
- Regular `apt update && apt upgrade` is the single most impactful security action you can take
- Security is a continuous process — review logs regularly and respond to anomalies

---

## Related Documents
- [Basic Linux CLI](basic-linux-cli.md)
- [Systemd File Configuration](systemd-file-configuration.md)
- [Network Security Basics](../Security/network-security-basics.md)
