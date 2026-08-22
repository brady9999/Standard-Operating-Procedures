# Active Directory Password & Group Policy
> A guide to configuring password policies, account lockout policies, and Group Policy Objects in Active Directory.

**Category:** Windows  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **GPO** | Group Policy Object | A collection of settings applied to users or computers in an OU |
| **GPMC** | Group Policy Management Console | The GUI tool for creating and managing GPOs |
| **Default Domain Policy** | Default Domain Policy | The built-in GPO that applies to all users and computers in the domain |
| **Password Policy** | Password Policy | Rules governing password complexity, length, age, and history |
| **Account Lockout Policy** | Account Lockout Policy | Rules governing when and how accounts get locked after failed logins |
| **Fine-Grained Password Policy** | FGPP | Allows different password policies for specific users or groups |
| **PSO** | Password Settings Object | The AD object used to apply fine-grained password policies |
| **Link** | GPO Link | Connecting a GPO to a site, domain, or OU so it applies |
| **Inheritance** | Policy Inheritance | Child OUs inherit GPOs from parent OUs by default |
| **Blocking Inheritance** | Block Inheritance | Preventing a child OU from inheriting parent GPO settings |
| **Enforced** | Enforced GPO | A GPO that cannot be blocked by child OUs |
| **WMI Filter** | WMI Filter | A query that limits which computers a GPO applies to |
| **Loopback Processing** | Loopback Processing | Applies user policies based on the computer's location rather than the user's |
| **gpupdate** | Group Policy Update | Command to force an immediate refresh of Group Policy |

---

## Overview
Group Policy allows IT administrators to centrally manage settings for users and computers across the entire domain. A single GPO can configure hundreds of settings on thousands of computers simultaneously — from password requirements to software installation to desktop wallpaper.

Password Policy and Account Lockout Policy are the most critical security settings in any AD environment. They live in the **Default Domain Policy** GPO and apply to every user in the domain.

---

## Prerequisites
- Domain Controller with AD DS installed
- Group Policy Management Console (GPMC) installed
- Domain Admin rights

---

## How to Open Group Policy Management Console

**Method 1 — Server Manager:**
1. Open **Server Manager**
2. Click **Tools** → **Group Policy Management**

**Method 2 — Run Dialog:**
1. Press **Windows + R**
2. Type `gpmc.msc`
3. Press **Enter**

---

## Understanding the GPMC Interface

```
Group Policy Management
└── Forest: company.com
    ├── Domains
    │   └── company.com
    │       ├── Default Domain Policy    ← Applies to entire domain
    │       ├── Domain Controllers       ← OU for DCs
    │       │   └── Default DC Policy   ← Applies to DCs only
    │       ├── IT                       ← Custom OU
    │       │   └── IT Security Policy  ← Custom GPO linked to IT OU
    │       └── Users
    ├── Sites
    └── Group Policy Objects             ← All GPOs stored here
```

- **Left panel** — domain and OU tree with linked GPOs
- **Right panel** — details, settings, and links for the selected GPO
- GPOs must be **linked** to an OU to apply to objects in that OU

---

## 1. Configuring Password Policy

Password policy applies domain-wide through the **Default Domain Policy**.

### GUI — Step by Step
1. Open **Group Policy Management**
2. Expand **Domains** → **company.com**
3. Right-click **Default Domain Policy** → **Edit**
4. The **Group Policy Management Editor** opens
5. Navigate to:
   `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`
6. Configure each setting by double-clicking:

| Setting | Recommended Value | What It Does |
|---------|-------------------|--------------|
| **Enforce password history** | 24 passwords | Prevents reusing recent passwords |
| **Maximum password age** | 90 days | Forces password change every 90 days |
| **Minimum password age** | 1 day | Prevents changing password immediately to cycle back to old one |
| **Minimum password length** | 12 characters | Minimum length requirement |
| **Password must meet complexity requirements** | Enabled | Requires uppercase, lowercase, numbers, and symbols |
| **Store passwords using reversible encryption** | Disabled | Security risk — leave disabled |

7. Double-click each setting → select **Define this policy setting** → set value → **OK**
8. Close the editor — policy applies at next Group Policy refresh

### PowerShell
```powershell
# View current password policy
Get-ADDefaultDomainPasswordPolicy

# Set password policy
Set-ADDefaultDomainPasswordPolicy -Identity "company.com" `
  -PasswordHistoryCount 24 `
  -MaxPasswordAge "90.00:00:00" `
  -MinPasswordAge "1.00:00:00" `
  -MinPasswordLength 12 `
  -ComplexityEnabled $true `
  -ReversibleEncryptionEnabled $false
```

---

## 2. Configuring Account Lockout Policy

### GUI — Step by Step
1. Open **Default Domain Policy** for editing (same as above)
2. Navigate to:
   `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy`
3. Configure:

| Setting | Recommended Value | What It Does |
|---------|-------------------|--------------|
| **Account lockout threshold** | 5 invalid attempts | Locks account after 5 wrong passwords |
| **Account lockout duration** | 30 minutes | How long account stays locked (0 = forever until admin unlocks) |
| **Reset account lockout counter after** | 30 minutes | Resets the failed attempt counter after this time |

4. Double-click each → **Define this policy setting** → set value → **OK**

### PowerShell
```powershell
# View current lockout policy
Get-ADDefaultDomainPasswordPolicy | Select-Object LockoutThreshold, LockoutDuration, LockoutObservationWindow

# Set lockout policy
Set-ADDefaultDomainPasswordPolicy -Identity "company.com" `
  -LockoutThreshold 5 `
  -LockoutDuration "00:30:00" `
  -LockoutObservationWindow "00:30:00"
```

---

## 3. Creating a New GPO

### GUI — Step by Step
1. Open **Group Policy Management**
2. Right-click the **OU** you want the GPO to apply to → **Create a GPO in this domain, and Link it here**
3. Enter a name e.g. `IT Department - Security Settings` → **OK**
4. The new GPO appears under the OU
5. Right-click the GPO → **Edit** to configure settings

### PowerShell
```powershell
# Create a new GPO
New-GPO -Name "IT Department - Security Settings" -Comment "Security settings for IT OU"

# Link the GPO to an OU
New-GPLink -Name "IT Department - Security Settings" `
  -Target "OU=IT,DC=company,DC=com"

# Create and link in one command
New-GPO -Name "IT-Security" | New-GPLink -Target "OU=IT,DC=company,DC=com"
```

---

## 4. Editing GPO Settings

### Common Settings Locations

**Computer Configuration → Policies → Windows Settings → Security Settings:**
- Account policies (password, lockout)
- Local policies (audit policy, user rights, security options)
- Windows Firewall settings
- Software restriction policies

**Computer Configuration → Policies → Administrative Templates:**
- Windows components settings
- System settings
- Network settings

**User Configuration → Policies → Administrative Templates:**
- Desktop settings
- Start menu settings
- Control Panel restrictions
- Browser settings

### Example — Disabling USB Storage via GPO
1. Edit your GPO
2. Navigate to: `Computer Configuration → Policies → Administrative Templates → System → Removable Storage Access`
3. Double-click **All Removable Storage classes: Deny all access**
4. Select **Enabled** → **OK**

### Example — Setting Desktop Wallpaper via GPO
1. Edit your GPO
2. Navigate to: `User Configuration → Policies → Administrative Templates → Desktop → Desktop`
3. Double-click **Desktop Wallpaper**
4. Select **Enabled**
5. Enter the path to the wallpaper file e.g. `\\server\wallpaper\company.jpg`
6. Click **OK**

---

## 5. Fine-Grained Password Policies

Fine-Grained Password Policies (FGPP) allow different password policies for specific users or groups — for example requiring admins to have longer passwords.

### GUI — Step by Step
1. Open **Active Directory Administrative Center** (`dsac.exe`)
2. In the left panel click your domain
3. Double-click **System** folder
4. Double-click **Password Settings Container**
5. In the right panel click **New** → **Password Settings**
6. Configure:
   - **Name** e.g. `Admin Password Policy`
   - **Precedence** — lower number = higher priority e.g. `10`
   - Set all password settings
7. Under **Directly Applies To** click **Add** → add the group or user
8. Click **OK**

### PowerShell
```powershell
# Create a Fine-Grained Password Policy
New-ADFineGrainedPasswordPolicy `
  -Name "Admin Password Policy" `
  -Precedence 10 `
  -MinPasswordLength 16 `
  -PasswordHistoryCount 24 `
  -ComplexityEnabled $true `
  -MaxPasswordAge "60.00:00:00" `
  -LockoutThreshold 3

# Apply it to a group
Add-ADFineGrainedPasswordPolicySubject `
  -Identity "Admin Password Policy" `
  -Subjects "Domain Admins"

# View all FGPPs
Get-ADFineGrainedPasswordPolicy -Filter *

# Check which policy applies to a specific user
Get-ADUserResultantPasswordPolicy -Identity "jsmith"
```

---

## 6. Forcing Group Policy Update

After making changes to a GPO you can force an immediate update rather than waiting for the automatic refresh (every 90 minutes for computers, at logon for users).

### On the Local Machine
```powershell
# Update all policies
gpupdate /force

# Update only computer policies
gpupdate /target:computer /force

# Update only user policies
gpupdate /target:user /force
```

### Remotely via GPMC
1. Open **Group Policy Management**
2. Right-click the OU → **Group Policy Update**
3. Click **Yes** — forces update on all computers in the OU

### Remotely via PowerShell
```powershell
# Force GP update on a remote computer
Invoke-GPUpdate -Computer "PC-NAME" -Force

# Force update on all computers in an OU
Get-ADComputer -Filter * -SearchBase "OU=IT,DC=company,DC=com" |
  ForEach-Object { Invoke-GPUpdate -Computer $_.Name -Force }
```

---

## 7. Viewing Resultant Set of Policy (RSoP)

RSoP shows you exactly which policies are being applied to a user or computer and where they come from.

### GUI — Step by Step
1. Open **Group Policy Management**
2. Right-click **Group Policy Results** → **Group Policy Results Wizard**
3. Select the computer and user to analyze
4. Click **Next** → **Finish**
5. A report shows all applied settings and their source GPO

### Command Line
```powershell
# Run RSoP for the current user and computer
rsop.msc

# Generate a text report
gpresult /r

# Generate an HTML report
gpresult /h "C:\GPReport.html"

# Check policies for a specific user on a remote computer
gpresult /s "PC-NAME" /u "COMPANY\jsmith" /r
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| GPO not applying | GPO not linked to the correct OU | Check links in GPMC |
| GPO applying to wrong users | Security filtering not configured | Check Security Filtering on the GPO — remove Authenticated Users if needed |
| Password policy not working | FGPP overriding domain policy | Check FGPP precedence and which users it applies to |
| Settings reverting after reboot | GPO conflict with another policy | Use RSoP to find which GPO wins |
| gpupdate fails | WMI service not running | Start the WMI service: `Start-Service winmgmt` |
| Account lockouts happening constantly | Cached credentials on old device | Check Event Viewer Security log for Event ID 4740 to find source |

---

## Quick Reference

```powershell
# View domain password policy
Get-ADDefaultDomainPasswordPolicy

# Force GP update locally
gpupdate /force

# View applied policies
gpresult /r

# Generate HTML report
gpresult /h "C:\report.html"

# List all GPOs
Get-GPO -All

# Backup all GPOs
Backup-GPO -All -Path "C:\GPOBackup"

# Restore a GPO from backup
Restore-GPO -Name "Default Domain Policy" -Path "C:\GPOBackup"

# Check resultant password policy for a user
Get-ADUserResultantPasswordPolicy -Identity "username"
```

---

## Notes
- Password policy in the Default Domain Policy applies to ALL users — use Fine-Grained Password Policies for exceptions
- Never delete the Default Domain Policy or Default Domain Controllers Policy — edit them carefully instead
- Always backup GPOs before making significant changes
- GPO changes can take up to 90 minutes to apply automatically — use `gpupdate /force` to apply immediately
- Use **Enforced** sparingly — it overrides all child OU policies and can cause unintended consequences

---

## Related Documents
- [Active Directory User Management](active-directory-user-management.md)
- [Active Directory Domain Configuration](active-directory-domain-configuration.md)
- [Windows Event Viewer](windows-event-viewer.md)
