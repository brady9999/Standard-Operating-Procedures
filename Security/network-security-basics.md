# Network Security Basics
> A guide to fundamental network security concepts, practices, and tools for IT professionals.

**Category:** Security  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **CIA Triad** | Confidentiality, Integrity, Availability | The three core principles of information security |
| **Threat** | Security Threat | Any potential danger to an information system |
| **Vulnerability** | Vulnerability | A weakness that can be exploited by a threat |
| **Risk** | Security Risk | The potential for loss when a threat exploits a vulnerability |
| **Attack Vector** | Attack Vector | The path or method used by an attacker to gain access |
| **Firewall** | Firewall | A system that monitors and controls network traffic based on rules |
| **IDS** | Intrusion Detection System | Monitors network traffic and alerts on suspicious activity |
| **IPS** | Intrusion Prevention System | Like IDS but actively blocks suspicious traffic |
| **SIEM** | Security Information and Event Management | Collects and analyzes security logs from across the environment |
| **SOC** | Security Operations Center | A team responsible for monitoring and responding to security events |
| **Zero Trust** | Zero Trust Architecture | A security model that trusts nothing by default — verify everything |
| **MFA** | Multi-Factor Authentication | Requiring two or more forms of verification to log in |
| **SSO** | Single Sign-On | One login grants access to multiple systems |
| **ACL** | Access Control List | Rules defining who can access what resources |
| **RBAC** | Role-Based Access Control | Access granted based on a user's role, not their identity |
| **Least Privilege** | Principle of Least Privilege | Users and systems should have only the minimum access needed |
| **Defense in Depth** | Defense in Depth | Multiple layers of security so no single failure is catastrophic |
| **Patch** | Security Patch | An update that fixes a known vulnerability |
| **CVE** | Common Vulnerabilities and Exposures | A public database of known security vulnerabilities |
| **CVSS** | Common Vulnerability Scoring System | A score from 0-10 rating the severity of a vulnerability |
| **Pen Test** | Penetration Test | An authorized simulated attack to find vulnerabilities |
| **Social Engineering** | Social Engineering | Manipulating people into revealing information or taking actions |
| **Phishing** | Phishing | A social engineering attack via fake emails or websites |
| **Ransomware** | Ransomware | Malware that encrypts files and demands payment for decryption |
| **DDoS** | Distributed Denial of Service | Overwhelming a system with traffic to make it unavailable |
| **MITM** | Man in the Middle | Intercepting communication between two parties |
| **DNS Poisoning** | DNS Cache Poisoning | Corrupting DNS records to redirect traffic to malicious sites |
| **ARP Spoofing** | ARP Spoofing | Sending fake ARP messages to intercept traffic on a LAN |

---

## Overview — The CIA Triad

Every security decision should be evaluated against three core principles:

**Confidentiality** — Ensuring information is only accessible to authorized parties
- Example: Encrypting data, access controls, MFA

**Integrity** — Ensuring information is accurate and has not been tampered with
- Example: File hashing, digital signatures, audit logs

**Availability** — Ensuring systems and data are accessible when needed
- Example: Redundancy, backups, DDoS protection, UPS systems

---

## 1. Defense in Depth

Never rely on a single security control. Layer multiple defenses so if one fails others remain.

```
Layer 1 — Physical Security
  - Locked server rooms, badge access, cameras

Layer 2 — Network Perimeter
  - Firewalls, IDS/IPS, DMZ

Layer 3 — Network Segmentation
  - VLANs, subnets, ACLs

Layer 4 — Endpoint Security
  - Antivirus, EDR, patch management, host firewall

Layer 5 — Application Security
  - Web application firewalls, secure coding, patching

Layer 6 — Data Security
  - Encryption at rest and in transit, DLP, backups

Layer 7 — Identity and Access
  - MFA, least privilege, RBAC, AD policies

Layer 8 — Monitoring and Response
  - SIEM, SOC, incident response plan
```

---

## 2. Firewall Configuration Basics

Firewalls control what traffic can enter and leave your network.

### Types of Firewalls

| Type | How It Works |
|------|-------------|
| **Packet Filter** | Inspects headers only — source/dest IP and port |
| **Stateful** | Tracks connection state — knows if traffic is part of an established session |
| **Application Layer (Layer 7)** | Inspects actual content — can block specific applications |
| **Next-Generation (NGFW)** | Combines all above plus IPS, SSL inspection, and application awareness |

### Windows Firewall — PowerShell
```powershell
# View all firewall rules
Get-NetFirewallRule | Select-Object DisplayName, Enabled, Direction, Action

# Allow a specific port inbound
New-NetFirewallRule -DisplayName "Allow RDP" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 3389 `
  -Action Allow

# Block a port
New-NetFirewallRule -DisplayName "Block Telnet" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 23 `
  -Action Block

# Enable/disable the firewall
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# View firewall profiles
Get-NetFirewallProfile

# Block all inbound by default
Set-NetFirewallProfile -Profile Public -DefaultInboundAction Block
```

### Cisco ACL as Firewall (Router)
```ios
! Block telnet inbound on WAN interface
RTR-MAIN-01(config)# ip access-list extended WAN-INBOUND
RTR-MAIN-01(config-ext-nacl)# deny tcp any any eq 23 log
RTR-MAIN-01(config-ext-nacl)# deny tcp any any eq 135 log
RTR-MAIN-01(config-ext-nacl)# permit ip any any
RTR-MAIN-01(config-ext-nacl)# exit

RTR-MAIN-01(config)# interface GigabitEthernet 0/1
RTR-MAIN-01(config-if)# ip access-group WAN-INBOUND in
```

---

## 3. Network Segmentation

Segmenting your network limits the blast radius of a security incident — if one segment is compromised others remain protected.

### VLAN Segmentation Strategy
```
VLAN 10  — Corporate Users       (trusted)
VLAN 20  — Servers               (restricted — only necessary ports open)
VLAN 30  — Guest WiFi            (internet only — no LAN access)
VLAN 40  — IoT Devices           (isolated — no access to other VLANs)
VLAN 50  — VoIP                  (QoS priority — isolated from data)
VLAN 99  — Management            (restricted to IT staff only)
```

### ACL Between VLANs
```ios
! Prevent guest VLAN from accessing corporate VLAN
RTR-MAIN-01(config)# ip access-list extended GUEST-RESTRICT
RTR-MAIN-01(config-ext-nacl)# deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
RTR-MAIN-01(config-ext-nacl)# deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
RTR-MAIN-01(config-ext-nacl)# permit ip 192.168.30.0 0.0.0.255 any
RTR-MAIN-01(config-ext-nacl)# exit

RTR-MAIN-01(config)# interface vlan 30
RTR-MAIN-01(config-if)# ip access-group GUEST-RESTRICT in
```

---

## 4. Password and Authentication Security

### Password Policy Best Practices
- Minimum 12 characters
- Complexity — uppercase, lowercase, numbers, symbols
- No reuse of last 24 passwords
- Maximum age 90 days
- Account lockout after 5 failed attempts

### Multi-Factor Authentication
Always enable MFA for:
- VPN access
- Remote desktop
- Email (especially admin accounts)
- Cloud services (Azure, AWS)
- Any internet-facing login

### PowerShell — Check Password Policy
```powershell
# View domain password policy
Get-ADDefaultDomainPasswordPolicy

# Check when a user's password expires
Get-ADUser -Identity "jsmith" -Properties PasswordExpired, PasswordLastSet |
  Select-Object Name, PasswordExpired, PasswordLastSet

# Find users with password never expires
Get-ADUser -Filter {PasswordNeverExpires -eq $true} |
  Select-Object Name, SamAccountName
```

---

## 5. Patch Management

Unpatched systems are one of the most common attack vectors. Establish a regular patching schedule.

### Windows Update — PowerShell
```powershell
# Check for available updates
Get-WindowsUpdate

# Install all available updates
Install-WindowsUpdate -AcceptAll -AutoReboot

# View installed updates
Get-HotFix | Sort-Object InstalledOn -Descending

# Check specific KB
Get-HotFix -Id KB5012345

# View pending reboots
Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired"
```

### Patch Management Best Practices
```
1. Test patches in a dev/test environment first
2. Deploy to non-critical systems first
3. Monitor for issues — have a rollback plan
4. Document all patch activity
5. Target patching within 30 days for critical CVEs
6. Critical severity (CVSS 9.0+) — patch within 24-72 hours
```

---

## 6. Audit Logging and Monitoring

You cannot defend what you cannot see. Logging everything is critical.

### Windows Audit Policy — PowerShell
```powershell
# View current audit policy
auditpol /get /category:*

# Enable logon auditing
auditpol /set /subcategory:"Logon" /success:enable /failure:enable

# Enable account management auditing
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable

# Enable object access auditing
auditpol /set /subcategory:"File System" /success:enable /failure:enable
```

### Key Events to Monitor

| Event ID | Description | Priority |
|----------|-------------|----------|
| 4625 | Failed login attempt | High |
| 4740 | Account locked out | High |
| 4720 | User account created | Medium |
| 4726 | User account deleted | High |
| 4728 | Member added to security group | High |
| 4756 | Member added to universal group | High |
| 4648 | Login with explicit credentials | Medium |
| 7045 | New service installed | High |
| 4698 | Scheduled task created | Medium |
| 4672 | Admin privileges assigned | High |

### PowerShell — Query Security Logs
```powershell
# Find all failed login attempts in the last 24 hours
$yesterday = (Get-Date).AddHours(-24)
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4625
  StartTime = $yesterday
} | Select-Object TimeCreated, Message

# Find account lockouts
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4740} |
  Select-Object TimeCreated, Message

# Find new users created
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4720} |
  Select-Object TimeCreated, Message
```

---

## 7. Encryption

### Encryption at Rest
- **BitLocker** — encrypts entire Windows drives
- **EFS** — encrypts individual files and folders
- **Database encryption** — encrypt sensitive database columns

```powershell
# Enable BitLocker on C drive
Enable-BitLocker -MountPoint "C:" `
  -EncryptionMethod Aes256 `
  -RecoveryPasswordProtector

# Check BitLocker status
Get-BitLockerVolume

# Back up recovery key to AD
Backup-BitLockerKeyProtector -MountPoint "C:" `
  -KeyProtectorId (Get-BitLockerVolume -MountPoint "C:").KeyProtector[0].KeyProtectorId
```

### Encryption in Transit
- Always use **HTTPS** (TLS 1.2 or 1.3) for web traffic
- Use **SSH** instead of Telnet
- Use **SFTP** instead of FTP
- Use **SNMPv3** instead of v1/v2c
- Use **LDAPS** instead of LDAP for Active Directory queries

### Checking TLS on a Server
```powershell
# Test TLS connection
Test-NetConnection -ComputerName "server.company.com" -Port 443

# Check what TLS versions are enabled
Get-TlsCipherSuite | Select-Object Name, Certificate
```

---

## 8. Common Attack Types and Prevention

### Phishing
**Attack:** Fake email tricks user into clicking a malicious link or revealing credentials
**Prevention:**
- Email filtering (SPF, DKIM, DMARC)
- Security awareness training
- MFA so stolen passwords alone aren't enough

### Ransomware
**Attack:** Malware encrypts files, demands payment for decryption
**Prevention:**
- Offline backups (3-2-1 rule)
- Endpoint protection / EDR
- Disable macros in Office documents
- Keep systems patched
- Network segmentation to limit spread

### Brute Force
**Attack:** Repeatedly trying passwords until one works
**Prevention:**
- Account lockout policy
- MFA
- Complex passwords
- Rate limiting

### Man in the Middle (MITM)
**Attack:** Attacker positions themselves between client and server to intercept traffic
**Prevention:**
- Use encryption (HTTPS, SSH, VPN)
- Certificate pinning
- HSTS (HTTP Strict Transport Security)

### ARP Spoofing
**Attack:** Sending fake ARP responses to redirect LAN traffic through the attacker
**Prevention:**
- Dynamic ARP Inspection (DAI) on Cisco switches
- Network monitoring for ARP anomalies
- VLANs to segment broadcast domains

### DNS Poisoning
**Attack:** Corrupting DNS cache to redirect users to malicious sites
**Prevention:**
- DNSSEC
- Use trusted DNS servers
- Monitor DNS for unusual records

---

## 9. Security Hardening Checklist

### Windows Server Hardening
```powershell
# Disable unnecessary services
Stop-Service -Name "Spooler" -Force    # Disable print spooler on non-print servers
Set-Service -Name "Spooler" -StartupType Disabled

# Disable SMBv1 (vulnerable)
Set-SmbServerConfiguration -EnableSMB1Protocol $false

# Enable Windows Firewall
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# Disable guest account
Disable-LocalUser -Name "Guest"

# Rename administrator account
Rename-LocalUser -Name "Administrator" -NewName "LocalAdmin"

# Enable audit logging
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
```

### Cisco Switch Hardening
```ios
! Disable unused services
no service finger
no service udp-small-servers
no service tcp-small-servers
no ip http server
no ip bootp server

! Disable CDP on edge ports
interface range FastEthernet 0/1-24
 no cdp enable

! Enable port security
interface range FastEthernet 0/1-24
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict

! Set SSH only
line vty 0 15
 transport input ssh
```

---

## 10. Incident Response — Basic Framework

When a security incident is detected follow these steps:

```
1. IDENTIFY
   - Detect and confirm the incident
   - Determine scope — what systems are affected?

2. CONTAIN
   - Isolate affected systems from the network
   - Disable compromised accounts
   - Preserve evidence (do not power off — memory forensics)

3. ERADICATE
   - Remove malware or threat actor access
   - Patch the vulnerability that was exploited
   - Reset compromised credentials

4. RECOVER
   - Restore systems from clean backups
   - Verify integrity before bringing back online
   - Monitor for reinfection

5. LESSONS LEARNED
   - Document what happened and how
   - Update security controls to prevent recurrence
   - Report to management and relevant parties
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Accounts getting locked out constantly | Brute force attack or cached credentials | Check Event ID 4740 source, review lockout policy |
| Ransomware spreading across network | No segmentation or backup | Isolate affected systems, restore from offline backup |
| Phishing emails getting through | No email filtering | Implement SPF, DKIM, DMARC, and email gateway filtering |
| System compromised via unpatched vulnerability | Missing patches | Deploy patch management, prioritize critical CVEs |
| Unauthorized access to sensitive data | Excessive permissions | Audit permissions, apply least privilege |

---

## Quick Reference

```powershell
# View firewall rules
Get-NetFirewallRule | Where-Object Enabled -eq True

# Check for failed logins
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} -MaxEvents 50

# Check BitLocker status
Get-BitLockerVolume

# Find accounts with password never expires
Get-ADUser -Filter {PasswordNeverExpires -eq $true}

# View installed updates
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 20

# Disable SMBv1
Set-SmbServerConfiguration -EnableSMB1Protocol $false
```

---

## Notes
- Security is never finished — it requires continuous monitoring and improvement
- The human element is the weakest link — regular security awareness training is essential
- 3-2-1 backup rule: 3 copies of data, 2 different media types, 1 offsite
- Always assume breach — design systems assuming the attacker is already inside
- Document everything — incident response, changes, and decisions
- Never security through obscurity alone — assume attackers will figure out your system

---

## Related Documents
- [ISO 27001 Basics](iso-27001-basics.md)
- [Active Directory Password & Group Policy](../Windows/active-directory-password-local--group-policy.md)
- [Windows Event Viewer](../Windows/windows-event-viewer.md)
- [VLAN Configuration](../Networking/vlan-configuration-cisco-switch.md)
