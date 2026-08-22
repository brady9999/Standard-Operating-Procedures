# Microsoft 365 User Management
> A guide to managing users, licenses, groups, and settings in Microsoft 365 Admin Center and PowerShell.

**Category:** Microsoft  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **M365** | Microsoft 365 | Microsoft's cloud productivity suite — formerly Office 365 |
| **Admin Center** | Microsoft 365 Admin Center | The web portal for managing M365 users and settings |
| **Tenant** | M365 Tenant | Your organization's instance of Microsoft 365 |
| **UPN** | User Principal Name | The user's login name e.g. brady@company.com |
| **License** | M365 License | A subscription that grants access to M365 apps and services |
| **SKU** | Stock Keeping Unit | The product identifier for a license type |
| **MFA** | Multi-Factor Authentication | Requiring a second verification step to log in |
| **SSPR** | Self-Service Password Reset | Allows users to reset their own passwords |
| **Exchange Online** | Exchange Online | The cloud email service included in M365 |
| **SharePoint** | SharePoint Online | The cloud document storage and collaboration platform |
| **OneDrive** | OneDrive for Business | Personal cloud storage for each M365 user |
| **Teams** | Microsoft Teams | The collaboration and video conferencing platform |
| **Entra ID** | Microsoft Entra ID | Formerly Azure AD — the identity platform behind M365 |
| **Global Admin** | Global Administrator | The highest-privilege admin role in M365 |
| **Guest User** | Guest User | An external user invited to collaborate |
| **Distribution Group** | Distribution Group | An email list that forwards to multiple recipients |
| **Security Group** | Security Group | A group used to manage access to resources |
| **M365 Group** | Microsoft 365 Group | A group with shared mailbox, calendar, SharePoint, and Teams |

---

## Overview
Microsoft 365 is a cloud-based productivity suite. As an IT administrator you'll manage user accounts, assign licenses, configure security settings, and troubleshoot access issues. Most management tasks can be done through the Admin Center web portal or PowerShell.

**Admin Center URL:** https://admin.microsoft.com

---

## 1. Accessing the Admin Center

### GUI — Step by Step
1. Go to **https://admin.microsoft.com**
2. Sign in with a Global Admin or User Admin account
3. The **Home** page shows a summary of your tenant
4. Left navigation:
   - **Users** — manage user accounts
   - **Groups** — manage distribution, security, and M365 groups
   - **Billing** — manage licenses and subscriptions
   - **Settings** — tenant-wide configuration
   - **Exchange** — email settings
   - **SharePoint** — SharePoint administration
   - **Teams** — Teams administration

---

## 2. Creating a New User

### GUI — Step by Step
1. Go to **Admin Center** → **Users** → **Active Users**
2. Click **Add a user**
3. Fill in:
   - **First name** and **Last name**
   - **Display name** — how they appear in Teams and email
   - **Username** — becomes their UPN e.g. `jsmith@company.com`
4. Click **Next**
5. Set **Location** (country — required for licensing)
6. Assign a **License** — check the appropriate M365 plan
7. Click **Next**
8. Assign **Admin roles** if needed (leave unchecked for standard users)
9. Set a **Password** — auto-generated or custom
10. Choose whether the user must change password at first login
11. Click **Next** → review → **Finish adding**

### PowerShell
```powershell
# Connect to Microsoft 365
Connect-MsolService
# Or using newer module:
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Create a new user
New-MsolUser `
  -DisplayName "John Smith" `
  -FirstName "John" `
  -LastName "Smith" `
  -UserPrincipalName "jsmith@company.com" `
  -Password "TempPass123!" `
  -ForceChangePassword $true `
  -UsageLocation "CA"

# Assign a license
Set-MsolUserLicense `
  -UserPrincipalName "jsmith@company.com" `
  -AddLicenses "company:ENTERPRISEPACK"

# View available license SKUs
Get-MsolAccountSku
```

---

## 3. Managing User Licenses

### GUI — Assign License
1. Go to **Active Users** → click the user
2. Click the **Licenses and Apps** tab
3. Check the license to assign
4. Expand the license to enable/disable specific apps
5. Click **Save changes**

### GUI — Remove License
1. Same as above — uncheck the license
2. Click **Save changes**

### PowerShell
```powershell
# View all license SKUs and availability
Get-MsolAccountSku | Select-Object AccountSkuId, ActiveUnits, ConsumedUnits

# Assign a license
Set-MsolUserLicense -UserPrincipalName "jsmith@company.com" `
  -AddLicenses "company:ENTERPRISEPACK"

# Remove a license
Set-MsolUserLicense -UserPrincipalName "jsmith@company.com" `
  -RemoveLicenses "company:ENTERPRISEPACK"

# View a user's current licenses
Get-MsolUser -UserPrincipalName "jsmith@company.com" | Select-Object Licenses

# Find all unlicensed users
Get-MsolUser -All | Where-Object {$_.IsLicensed -eq $false} |
  Select-Object DisplayName, UserPrincipalName

# Find all users with a specific license
Get-MsolUser -All | Where-Object {$_.Licenses.AccountSkuId -like "*ENTERPRISEPACK*"} |
  Select-Object DisplayName, UserPrincipalName
```

---

## 4. Resetting Passwords

### GUI — Step by Step
1. Go to **Active Users** → click the user
2. Click **Reset password** (key icon)
3. Choose auto-generated or custom password
4. Choose whether to require password change at next login
5. Optionally send password to an email address
6. Click **Reset password**

### PowerShell
```powershell
# Reset password
Set-MsolUserPassword `
  -UserPrincipalName "jsmith@company.com" `
  -NewPassword "NewPass123!" `
  -ForceChangePassword $true

# Reset to auto-generated password
$result = Set-MsolUserPassword `
  -UserPrincipalName "jsmith@company.com" `
  -ForceChangePassword $true
$result.Password
```

---

## 5. Blocking and Unblocking Sign-In

### GUI — Step by Step
1. Go to **Active Users** → click the user
2. Click **Block sign-in** (block icon)
3. Check **Block this user from signing in**
4. Click **Save changes**

### PowerShell
```powershell
# Block a user
Set-MsolUser -UserPrincipalName "jsmith@company.com" -BlockCredential $true

# Unblock a user
Set-MsolUser -UserPrincipalName "jsmith@company.com" -BlockCredential $false

# Check if user is blocked
Get-MsolUser -UserPrincipalName "jsmith@company.com" | Select-Object BlockCredential
```

---

## 6. Deleting and Restoring Users

### GUI — Delete User
1. Go to **Active Users** → check the box next to the user
2. Click **Delete user**
3. Confirm — the user moves to the **Deleted Users** list for 30 days

### GUI — Restore Deleted User
1. Go to **Users** → **Deleted Users**
2. Check the user → **Restore user**
3. The user is restored with their original UPN and license

### PowerShell
```powershell
# Delete a user
Remove-MsolUser -UserPrincipalName "jsmith@company.com"

# View deleted users (soft-deleted — recoverable for 30 days)
Get-MsolUser -ReturnDeletedUsers | Select-Object DisplayName, UserPrincipalName

# Restore a deleted user
Restore-MsolUser -UserPrincipalName "jsmith@company.com"

# Permanently delete (removes from recycle bin immediately)
Remove-MsolUser -UserPrincipalName "jsmith@company.com" -RemoveFromRecycleBin
```

---

## 7. Managing Groups

### Group Types

| Type | Purpose | Has Email? |
|------|---------|-----------|
| **Distribution Group** | Email list — forwards to all members | Yes |
| **Security Group** | Access control — control who can access resources | No |
| **Mail-Enabled Security Group** | Both email list and access control | Yes |
| **Microsoft 365 Group** | Full collaboration — email, calendar, SharePoint, Teams | Yes |

### GUI — Create a Group
1. Go to **Groups** → **Active Groups**
2. Click **Add a group**
3. Select group type
4. Enter name, description
5. Set owners and members
6. Configure email address (if applicable)
7. Click **Create group**

### PowerShell
```powershell
# Create a security group
New-MsolGroup -DisplayName "IT-Admins" -Description "IT Administrator Group"

# Add member to group
Add-MsolGroupMember -GroupObjectId (Get-MsolGroup -SearchString "IT-Admins").ObjectId `
  -GroupMemberObjectId (Get-MsolUser -UserPrincipalName "jsmith@company.com").ObjectId

# View group members
Get-MsolGroupMember -GroupObjectId (Get-MsolGroup -SearchString "IT-Admins").ObjectId

# Remove member from group
Remove-MsolGroupMember -GroupObjectId (Get-MsolGroup -SearchString "IT-Admins").ObjectId `
  -GroupMemberObjectId (Get-MsolUser -UserPrincipalName "jsmith@company.com").ObjectId
```

---

## 8. Multi-Factor Authentication (MFA)

### GUI — Enable MFA for a User
1. Go to **Admin Center** → **Users** → **Active Users**
2. Click **Multi-factor authentication** at the top
3. Find the user → check their box
4. Click **Enable** on the right panel
5. Click **Enable multi-factor auth** to confirm

### GUI — Enforce MFA (Forces setup at next login)
- Same as above but click **Enforce** instead of Enable

### PowerShell
```powershell
# Enable MFA for a user
$mfaSettings = New-Object -TypeName Microsoft.Online.Administration.StrongAuthenticationRequirement
$mfaSettings.RelyingParty = "*"
$mfaSettings.State = "Enabled"

Set-MsolUser -UserPrincipalName "jsmith@company.com" `
  -StrongAuthenticationRequirements $mfaSettings

# Check MFA status for all users
Get-MsolUser -All | Select-Object DisplayName, UserPrincipalName,
  @{N="MFAState"; E={$_.StrongAuthenticationRequirements.State}} |
  Where-Object {$_.MFAState -ne $null}

# Find users without MFA
Get-MsolUser -All | Where-Object {$_.StrongAuthenticationRequirements.Count -eq 0} |
  Select-Object DisplayName, UserPrincipalName
```

---

## 9. Shared Mailboxes

Shared mailboxes allow multiple users to read and send from a common email address without a license.

### GUI — Create Shared Mailbox
1. Go to **Admin Center** → **Exchange** → **Recipients** → **Mailboxes**
2. Click **Add a shared mailbox**
3. Enter name and email address
4. Add members who can access it
5. Click **Create**

### PowerShell
```powershell
# Connect to Exchange Online
Connect-ExchangeOnline -UserPrincipalName admin@company.com

# Create shared mailbox
New-Mailbox -Shared -Name "IT Support" -DisplayName "IT Support" `
  -Alias "itsupport" -PrimarySmtpAddress "itsupport@company.com"

# Grant access to the shared mailbox
Add-MailboxPermission -Identity "itsupport@company.com" `
  -User "jsmith@company.com" `
  -AccessRights FullAccess `
  -InheritanceType All

# Grant send-as permission
Add-RecipientPermission -Identity "itsupport@company.com" `
  -Trustee "jsmith@company.com" `
  -AccessRights SendAs
```

---

## 10. User Offboarding Checklist

```powershell
# 1. Block sign-in immediately
Set-MsolUser -UserPrincipalName "leavinguser@company.com" -BlockCredential $true

# 2. Reset password to random value
Set-MsolUserPassword -UserPrincipalName "leavinguser@company.com" `
  -NewPassword ([System.Web.Security.Membership]::GeneratePassword(20, 5))

# 3. Revoke all active sessions
Revoke-AzureADUserAllRefreshToken -ObjectId (Get-AzureADUser -ObjectId "leavinguser@company.com").ObjectId

# 4. Remove from all groups
Get-MsolGroup -All | ForEach-Object {
  $members = Get-MsolGroupMember -GroupObjectId $_.ObjectId
  if ($members.EmailAddress -contains "leavinguser@company.com") {
    Remove-MsolGroupMember -GroupObjectId $_.ObjectId `
      -GroupMemberObjectId (Get-MsolUser -UserPrincipalName "leavinguser@company.com").ObjectId
  }
}

# 5. Convert mailbox to shared (preserves email without license)
# In Exchange Admin Center: Recipients → Mailboxes → select user → Convert to shared mailbox

# 6. Remove license (do this AFTER converting mailbox)
Set-MsolUserLicense -UserPrincipalName "leavinguser@company.com" `
  -RemoveLicenses "company:ENTERPRISEPACK"

# 7. Delete user (goes to recycle bin for 30 days)
Remove-MsolUser -UserPrincipalName "leavinguser@company.com"
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| User can't sign in | Account blocked or MFA issue | Check block status, reset MFA |
| License not applying | No usage location set | Set location: `Set-MsolUser -UsageLocation "CA"` |
| Can't assign license | No licenses available | Check billing — purchase more licenses |
| MFA not prompting | MFA not enabled | Enable MFA in Admin Center |
| Email not working | License missing Exchange | Verify license includes Exchange Online |
| Teams not available | License doesn't include Teams | Upgrade license plan |

---

## Quick Reference

```powershell
# Connect
Connect-MsolService

# List all users
Get-MsolUser -All | Select-Object DisplayName, UserPrincipalName, IsLicensed

# Create user
New-MsolUser -DisplayName "Name" -UserPrincipalName "user@domain.com" -UsageLocation "CA"

# Assign license
Set-MsolUserLicense -UserPrincipalName "user@domain.com" -AddLicenses "tenant:SKU"

# Block sign-in
Set-MsolUser -UserPrincipalName "user@domain.com" -BlockCredential $true

# Reset password
Set-MsolUserPassword -UserPrincipalName "user@domain.com" -NewPassword "Pass123!"

# Delete user
Remove-MsolUser -UserPrincipalName "user@domain.com"

# Restore user
Restore-MsolUser -UserPrincipalName "user@domain.com"

# View license SKUs
Get-MsolAccountSku
```

---

## Notes
- Always set **UsageLocation** before assigning licenses — it's required for compliance reasons
- The **Global Admin** role has unrestricted access — assign it sparingly
- Deleted users are recoverable for **30 days** — after that they're permanently deleted
- Shared mailboxes don't require a license as long as the mailbox is under 50GB
- MFA should be mandatory for all admin accounts — use Conditional Access policies for enforcement
- Use the **Microsoft 365 Admin Center mobile app** for quick tasks when away from your desk

---

## Related Documents
- [Entra ID Basics](entra-id-basics.md)
- [Intune Basics](intune-basics.md)
- [Active Directory User Management](../Windows/active-directory-user-management.md)
