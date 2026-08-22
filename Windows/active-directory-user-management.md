# Active Directory User Management
> Complete guide for managing users in an Active Directory environment using both GUI (ADUC) and PowerShell.

**Category:** Windows  
**Last Updated:** 2026-08-17  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **AD** | Active Directory | Microsoft's directory service for managing users, computers, and resources on a network |
| **DC** | Domain Controller | The server that runs Active Directory and handles authentication requests |
| **OU** | Organizational Unit | A folder-like container inside AD used to organize users, computers, and groups |
| **DN** | Distinguished Name | The full path to an object in AD e.g. `CN=John Smith,OU=IT,DC=company,DC=com` |
| **CN** | Common Name | The name of the object itself e.g. `CN=John Smith` |
| **SAM** | Security Account Manager | The username used to log in e.g. `jsmith` |
| **UPN** | User Principal Name | The email-style login name e.g. `jsmith@company.com` |
| **ADUC** | Active Directory Users and Computers | The GUI tool used to manage AD objects |
| **GPO** | Group Policy Object | A set of rules applied to users or computers in an OU |
| **LDAP** | Lightweight Directory Access Protocol | The protocol used to communicate with Active Directory |
| **Domain** | Domain | The logical grouping of all AD objects e.g. `company.com` |
| **Forest** | Forest | The top-level container that holds one or more domains |
| **Trust** | Trust Relationship | A link between two domains that allows users to access resources across them |
| **RSAT** | Remote Server Administration Tools | Tools installed on a workstation to manage AD without logging into the DC |
| **SID** | Security Identifier | A unique ID assigned to every user and group in AD |
| **AAD / Entra** | Azure Active Directory / Entra ID | Microsoft's cloud-based version of Active Directory |

---

## Overview
Active Directory (AD) is Microsoft's directory service used in enterprise environments to centrally manage users, computers, groups, and permissions. Instead of managing each computer individually, IT admins can control everything from one place — the Domain Controller.

When a user logs into a domain-joined computer, the computer sends their credentials to the DC which checks if they are who they say they are and what they are allowed to access. This is called **authentication**.

---

## Prerequisites
- Windows Server with Active Directory Domain Services (AD DS) installed
- An account with Domain Admin or Account Operator privileges
- Remote Server Administration Tools (RSAT) installed if managing from a workstation
- PowerShell with Active Directory module loaded

### Opening ADUC (Active Directory Users and Computers)
**Method 1 — Start Menu:**
1. Click **Start**
2. Search for **Active Directory Users and Computers**
3. Click to open

**Method 2 — Run Dialog:**
1. Press **Windows + R**
2. Type `dsa.msc`
3. Press **Enter**

**Method 3 — Server Manager:**
1. Open **Server Manager**
2. Click **Tools** in the top right
3. Select **Active Directory Users and Computers**

### Load the AD PowerShell Module
```powershell
Import-Module ActiveDirectory
```

---

## Understanding the ADUC Interface

When you open ADUC you will see a tree structure on the left side:

```
yourdomain.com
├── Builtin          ← Default built-in groups
├── Computers        ← Domain-joined computers
├── Domain Controllers  ← Your DC servers
├── ForeignSecurityPrincipals
└── Users            ← Default users container
    ├── IT           ← Example OU
    ├── HR           ← Example OU
    └── Disabled     ← Example OU for offboarded users
```

- **Right-click** on any OU to create new objects inside it
- **Double-click** any user to open their properties
- Use **Ctrl+F** or the Find button (binoculars icon) to search the entire directory

---

## 1. Creating a New User

### GUI — Step by Step
1. Open **ADUC** (`dsa.msc`)
2. In the left panel, navigate to the OU where the user should live (e.g. `Users > IT`)
3. Right-click the OU → **New** → **User**
4. A wizard will open. Fill in:
   - **First name** and **Last name**
   - **Full name** — auto-fills but can be edited
   - **User logon name** — this is the username e.g. `jsmith`
5. Click **Next**
6. Enter a temporary password in both fields
7. Check the following boxes as needed:
   - ✅ **User must change password at next logon** — recommended
   - ⬜ User cannot change password
   - ⬜ Password never expires
   - ⬜ Account is disabled
8. Click **Next** → review the summary → **Finish**
9. The new user will appear in the OU

### PowerShell
```powershell
New-ADUser `
  -Name "John Smith" `
  -GivenName "John" `
  -Surname "Smith" `
  -SamAccountName "jsmith" `
  -UserPrincipalName "jsmith@yourdomain.com" `
  -Path "OU=IT,OU=Users,DC=yourdomain,DC=com" `
  -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
  -ChangePasswordAtLogon $true `
  -Enabled $true
```

---

## 2. Resetting a User Password

### GUI — Step by Step
1. Open **ADUC**
2. Find the user — you can browse to their OU or use **Find** (Ctrl+F)
3. Right-click the user → **Reset Password**
4. A dialog box appears:
   - Enter the new password
   - Confirm the new password
   - Check **User must change password at next logon** if required
   - Check **Unlock the user's account** if they are also locked out
5. Click **OK**
6. A confirmation message will appear

### PowerShell
```powershell
Set-ADAccountPassword -Identity "jsmith" `
  -NewPassword (ConvertTo-SecureString "NewPass123!" -AsPlainText -Force) `
  -Reset

# Force password change at next logon
Set-ADUser -Identity "jsmith" -ChangePasswordAtLogon $true
```

---

## 3. Enabling and Disabling a User Account

### GUI — Step by Step
1. Open **ADUC**
2. Find the user
3. Right-click the user
4. Select **Disable Account** or **Enable Account**
   - A disabled account shows a downward arrow icon on the user
5. Click **OK** on the confirmation

### PowerShell
```powershell
# Disable
Disable-ADAccount -Identity "jsmith"

# Enable
Enable-ADAccount -Identity "jsmith"

# Check if account is enabled or disabled
Get-ADUser -Identity "jsmith" | Select-Object Name, Enabled
```

---

## 4. Unlocking a Locked Account

A user account locks out after too many failed login attempts. The lockout policy is set by Group Policy.

### GUI — Step by Step
1. Open **ADUC**
2. Find the user → right-click → **Properties**
3. Click the **Account** tab
4. If the account is locked you will see **Account is locked out** checkbox is checked
5. Uncheck **Account is locked out**
6. Click **Apply** → **OK**

> **Tip:** You can also right-click the user and select **Reset Password** — there is an option to unlock the account on the same dialog

### PowerShell
```powershell
# Unlock account
Unlock-ADAccount -Identity "jsmith"

# Check if account is locked
Get-ADUser -Identity "jsmith" -Properties LockedOut | Select-Object Name, LockedOut
```

---

## 5. Adding a User to a Group

### GUI — Method 1: Through the User
1. Open **ADUC**
2. Find the user → right-click → **Properties**
3. Click the **Member Of** tab
4. Click **Add**
5. Type the group name in the box
6. Click **Check Names** to verify it exists
7. Click **OK** → **Apply** → **OK**

### GUI — Method 2: Through the Group
1. Find the group in ADUC
2. Right-click → **Properties**
3. Click the **Members** tab
4. Click **Add**
5. Type the username → **Check Names** → **OK**

### PowerShell
```powershell
# Add one user to a group
Add-ADGroupMember -Identity "IT-Team" -Members "jsmith"

# Add multiple users at once
Add-ADGroupMember -Identity "IT-Team" -Members "jsmith","bjones","kwilliams"

# See what groups a user is in
Get-ADPrincipalGroupMembership -Identity "jsmith" | Select-Object Name
```

---

## 6. Removing a User from a Group

### GUI — Step by Step
1. Open **ADUC**
2. Find the user → right-click → **Properties**
3. Click the **Member Of** tab
4. Select the group you want to remove them from
5. Click **Remove**
6. Click **Apply** → **OK**

### PowerShell
```powershell
Remove-ADGroupMember -Identity "IT-Team" -Members "jsmith" -Confirm:$false
```

---

## 7. Modifying User Properties

### GUI — Step by Step
1. Open **ADUC**
2. Find the user → right-click → **Properties**
3. The Properties window has multiple tabs:
   - **General** — Display name, description, office, phone, email, website
   - **Address** — Street, city, province, postal code, country
   - **Account** — Logon name, logon hours, workstation restrictions, account expiry
   - **Profile** — Profile path, logon script, home folder
   - **Telephones** — Various phone numbers
   - **Organization** — Title, department, company, manager, direct reports
4. Make your changes
5. Click **Apply** → **OK**

### PowerShell
```powershell
Set-ADUser -Identity "jsmith" `
  -DisplayName "John Smith" `
  -Title "IT Technician" `
  -Department "Information Technology" `
  -OfficePhone "555-1234" `
  -EmailAddress "jsmith@yourdomain.com" `
  -Office "Winnipeg HQ"
```

---

## 8. Moving a User to a Different OU

### GUI — Step by Step
1. Open **ADUC**
2. Find the user
3. Right-click → **Move**
4. A dialog box shows the OU tree
5. Navigate to and select the destination OU
6. Click **OK**

> **Tip:** You can also drag and drop users between OUs in ADUC

### PowerShell
```powershell
Move-ADObject -Identity "CN=John Smith,OU=Users,DC=yourdomain,DC=com" `
  -TargetPath "OU=IT,OU=Staff,DC=yourdomain,DC=com"
```

---

## 9. Deleting a User Account

### GUI — Step by Step
1. Open **ADUC**
2. Find the user
3. Right-click → **Delete**
4. A confirmation dialog appears — click **Yes**

> ⚠️ **Best Practice:** Disable accounts before deleting. Deleted accounts cannot be recovered without AD Recycle Bin enabled. Move to a Disabled OU first and wait 30 days before permanently deleting.

### PowerShell
```powershell
# Disable first
Disable-ADAccount -Identity "jsmith"

# Then delete
Remove-ADUser -Identity "jsmith" -Confirm:$false
```

---

## 10. Finding Users in ADUC

### GUI — Using Find
1. Open **ADUC**
2. Click the **Find** button in the toolbar (binoculars icon) or press **Ctrl+F**
3. Make sure **Find** is set to **Users, Contacts, and Groups**
4. Make sure **In** is set to **Entire Directory**
5. Type the name or username in the **Name** field
6. Click **Find Now**
7. Results appear at the bottom — double-click to open

### PowerShell
```powershell
# Find a specific user by username
Get-ADUser -Identity "jsmith" -Properties *

# Search by name
Get-ADUser -Filter {Name -like "*Smith*"} -Properties DisplayName, EmailAddress

# Find all disabled accounts
Get-ADUser -Filter {Enabled -eq $false} | Select-Object Name, SamAccountName

# Find all locked accounts
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName

# Find users in a specific OU
Get-ADUser -Filter * -SearchBase "OU=IT,DC=yourdomain,DC=com"

# Find users who haven't logged in for 90 days
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff} -Properties LastLogonDate |
  Select-Object Name, LastLogonDate
```

---

## 11. User Onboarding Checklist

When a new employee joins:

**GUI Steps:**
1. Create user account in appropriate OU
2. Set temporary password with **User must change password at next logon**
3. Add to relevant groups via **Member Of** tab
4. Fill in **Organization** tab — title, department, manager
5. Confirm account is enabled

**PowerShell:**
```powershell
# Create the user
New-ADUser `
  -Name "Jane Doe" `
  -GivenName "Jane" `
  -Surname "Doe" `
  -SamAccountName "jdoe" `
  -UserPrincipalName "jdoe@yourdomain.com" `
  -Path "OU=IT,DC=yourdomain,DC=com" `
  -Title "IT Analyst" `
  -Department "Information Technology" `
  -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
  -ChangePasswordAtLogon $true `
  -Enabled $true

# Add to groups
Add-ADGroupMember -Identity "IT-Team" -Members "jdoe"
Add-ADGroupMember -Identity "VPN-Users" -Members "jdoe"
```

---

## 12. User Offboarding Checklist

When an employee leaves:

**GUI Steps:**
1. Disable the account — right-click → **Disable Account**
2. Reset password to something unknown
3. Remove from all groups via **Member Of** tab
4. Move to **Disabled** OU — right-click → **Move**
5. Update **Description** field with departure date

**PowerShell:**
```powershell
# 1. Disable the account
Disable-ADAccount -Identity "jsmith"

# 2. Reset to a random password
Set-ADAccountPassword -Identity "jsmith" `
  -NewPassword (ConvertTo-SecureString "Rand0mP@ss$(Get-Random)" -AsPlainText -Force) `
  -Reset

# 3. Remove from all groups except Domain Users
$user = Get-ADUser -Identity "jsmith" -Properties MemberOf
$user.MemberOf | ForEach-Object {
  Remove-ADGroupMember -Identity $_ -Members "jsmith" -Confirm:$false
}

# 4. Move to Disabled OU
Move-ADObject -Identity $user.DistinguishedName `
  -TargetPath "OU=Disabled,DC=yourdomain,DC=com"

# 5. Add departure note to description
Set-ADUser -Identity "jsmith" `
  -Description "Disabled $(Get-Date -Format 'yyyy-MM-dd') - Employee departed"
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Account keeps locking out | Bad cached credentials on a device | Check Event Viewer, clear cached credentials on all devices |
| User can't log in after password reset | Password doesn't meet complexity policy | Check domain password policy in Group Policy |
| Can't find user in ADUC | User may be in a different OU | Use Find (Ctrl+F) to search entire directory |
| PowerShell AD commands not found | AD module not loaded | Run `Import-Module ActiveDirectory` |
| Access denied running commands | Insufficient privileges | Run PowerShell as Domain Admin |
| User can log in but can't access resources | Not in correct security group | Check **Member Of** tab and add to appropriate groups |
| ADUC won't open | RSAT not installed on workstation | Install RSAT via Settings → Optional Features |

---

## Quick Reference — PowerShell Commands

```powershell
# Get all AD users
Get-ADUser -Filter *

# Get user details
Get-ADUser -Identity "jsmith" -Properties *

# Get user's group memberships
Get-ADPrincipalGroupMembership -Identity "jsmith" | Select-Object Name

# Check last logon time
Get-ADUser -Identity "jsmith" -Properties LastLogonDate | Select-Object Name, LastLogonDate

# Get password expiry info
Get-ADUser -Identity "jsmith" -Properties PasswordExpired, PasswordLastSet, PasswordNeverExpires

# List all OUs
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

# Bulk create users from CSV
Import-Csv "users.csv" | ForEach-Object {
  New-ADUser `
    -Name $_.Name `
    -SamAccountName $_.Username `
    -Enabled $true `
    -AccountPassword (ConvertTo-SecureString $_.Password -AsPlainText -Force)
}

# Export all users to CSV
Get-ADUser -Filter * -Properties DisplayName, EmailAddress, Department |
  Select-Object Name, SamAccountName, DisplayName, EmailAddress, Department |
  Export-Csv "ad-users.csv" -NoTypeInformation
```

---

## Notes
- Always disable accounts before deleting — gives you a recovery window
- Use descriptive OU structures to keep the directory organized
- Enable AD Recycle Bin in production environments for account recovery
- Document all changes — who was modified, when, and why
- Use least privilege — don't give Domain Admin rights unless absolutely necessary
- Group Policy (GPO) controls password complexity, lockout thresholds, and logon restrictions

---

## Related Documents
- [Windows Server Setup](#)
- [Group Policy Management](#)
- [LDAP Configuration](#)
- [User Onboarding Checklist](#)
