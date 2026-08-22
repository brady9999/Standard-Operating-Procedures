# Microsoft Entra ID Basics
> An introduction to Microsoft Entra ID (formerly Azure Active Directory) — cloud identity and access management.

**Category:** Microsoft  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Entra ID** | Microsoft Entra ID | Microsoft's cloud identity platform — formerly Azure Active Directory (Azure AD) |
| **Azure AD** | Azure Active Directory | The former name — still used in PowerShell and documentation |
| **Tenant** | Entra ID Tenant | Your organization's instance of Entra ID |
| **Directory** | Tenant Directory | The container holding all users, groups, and apps for the tenant |
| **Identity** | Identity | A representation of a person, service, or device that can be authenticated |
| **Authentication** | Authentication | Proving who you are — username/password, MFA |
| **Authorization** | Authorization | What you're allowed to do after authentication |
| **SSO** | Single Sign-On | One login grants access to multiple applications |
| **Token** | Access Token | A credential issued after authentication allowing access to resources |
| **App Registration** | App Registration | Registering an application to use Entra ID for authentication |
| **Service Principal** | Service Principal | An identity used by an application or service (not a human) |
| **Managed Identity** | Managed Identity | An automatically managed identity for Azure services — no passwords |
| **Conditional Access** | Conditional Access | Policies controlling access based on conditions (device compliance, location, risk) |
| **PIM** | Privileged Identity Management | Just-in-time privileged access — request admin roles only when needed |
| **Identity Protection** | Entra ID Protection | Detects and responds to identity-based risks |
| **Hybrid Identity** | Hybrid Identity | Synchronizing on-premises AD with Entra ID |
| **Entra Connect** | Microsoft Entra Connect | The tool that synchronizes on-premises AD to Entra ID |
| **SSPR** | Self-Service Password Reset | Users reset their own passwords without IT intervention |
| **B2B** | Business-to-Business | Inviting external users from other organizations |
| **B2C** | Business-to-Consumer | Identity management for customer-facing applications |
| **FIDO2** | FIDO2 | A passwordless authentication standard using hardware security keys |

---

## Overview
Microsoft Entra ID is the cloud identity platform that powers Microsoft 365, Azure, and thousands of third-party applications. It replaced Azure Active Directory (Azure AD) in branding but the underlying technology and PowerShell commands still reference "AzureAD."

**What Entra ID does:**
- Authenticates users signing into M365, Azure, and connected apps
- Manages user identities and access
- Enforces security policies (MFA, Conditional Access)
- Provides SSO to thousands of apps
- Synchronizes with on-premises Active Directory

**Entra ID vs On-Premises Active Directory:**

| Feature | On-Premises AD | Entra ID |
|---------|---------------|---------|
| Protocol | LDAP/Kerberos | OAuth 2.0/SAML/OpenID Connect |
| Location | Local domain controller | Microsoft cloud |
| Domain join | Traditional domain join | Entra ID join or hybrid join |
| Group Policy | Yes — via GPO | No GPO — use Intune |
| Primary use | Windows networks | Cloud apps and M365 |

---

## 1. Accessing Entra ID

**Admin Center URL:** https://entra.microsoft.com

Or access via **Azure Portal** → search **Microsoft Entra ID**

### Key Sections
```
Microsoft Entra ID
├── Overview              ← Tenant summary and quick stats
├── Users                 ← Manage user identities
├── Groups                ← Manage groups
├── Applications          ← App registrations and enterprise apps
├── Devices               ← Manage Entra ID joined devices
├── Conditional Access    ← Access policies
├── Identity Protection   ← Risk detection
├── Privileged Identity   ← PIM — just-in-time admin access
│   Management
└── Monitoring            ← Sign-in logs, audit logs
```

---

## 2. User Management in Entra ID

### GUI — Create a User
1. Go to **Entra ID** → **Users** → **New User** → **Create new user**
2. Fill in:
   - **User principal name** e.g. `jsmith@company.com`
   - **Display name** e.g. `John Smith`
   - **Password** — auto-generate or set manually
3. Optionally fill in **Properties** (job title, department, phone)
4. Optionally assign **Roles** (Global Admin, User Admin, etc.)
5. Click **Review + Create** → **Create**

### PowerShell
```powershell
# Install and connect
Install-Module AzureAD -Force
Connect-AzureAD

# Or using newer Microsoft Graph module
Install-Module Microsoft.Graph -Force
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Create a user
New-AzureADUser `
  -DisplayName "John Smith" `
  -GivenName "John" `
  -Surname "Smith" `
  -UserPrincipalName "jsmith@company.com" `
  -MailNickName "jsmith" `
  -PasswordProfile (New-Object Microsoft.Open.AzureAD.Model.PasswordProfile -Property @{
    Password = "TempPass123!"
    ForceChangePasswordNextLogin = $true
  }) `
  -AccountEnabled $true `
  -UsageLocation "CA"

# Get all users
Get-AzureADUser -All $true | Select-Object DisplayName, UserPrincipalName, AccountEnabled

# Get a specific user
Get-AzureADUser -ObjectId "jsmith@company.com"

# Update user properties
Set-AzureADUser -ObjectId "jsmith@company.com" `
  -Department "Information Technology" `
  -JobTitle "IT Technician" `
  -TelephoneNumber "204-555-1234"

# Disable a user account
Set-AzureADUser -ObjectId "jsmith@company.com" -AccountEnabled $false

# Delete a user
Remove-AzureADUser -ObjectId "jsmith@company.com"
```

---

## 3. Group Management

### Group Types in Entra ID

| Type | Membership | Use Case |
|------|-----------|---------|
| **Assigned** | Manually add/remove members | Small static groups |
| **Dynamic User** | Auto-based on user attributes (department, job title) | Auto-populated groups |
| **Dynamic Device** | Auto-based on device attributes (OS, compliance) | Device targeting for Intune |

### GUI — Create a Dynamic Group
1. Go to **Entra ID** → **Groups** → **New Group**
2. Set **Group type**: Security
3. Set **Membership type**: Dynamic User
4. Click **Add dynamic query**
5. Configure rules e.g.:
   - `department Equals "Information Technology"`
   - `jobTitle Contains "Manager"`
6. Click **Create**

### PowerShell
```powershell
# Create a security group
New-AzureADGroup `
  -DisplayName "IT-Administrators" `
  -MailEnabled $false `
  -SecurityEnabled $true `
  -MailNickName "IT-Administrators" `
  -Description "IT Administrator group"

# Add member to group
Add-AzureADGroupMember `
  -ObjectId (Get-AzureADGroup -SearchString "IT-Administrators").ObjectId `
  -RefObjectId (Get-AzureADUser -ObjectId "jsmith@company.com").ObjectId

# Get group members
Get-AzureADGroupMember -ObjectId (Get-AzureADGroup -SearchString "IT-Administrators").ObjectId

# Remove member from group
Remove-AzureADGroupMember `
  -ObjectId (Get-AzureADGroup -SearchString "IT-Administrators").ObjectId `
  -MemberId (Get-AzureADUser -ObjectId "jsmith@company.com").ObjectId
```

---

## 4. Admin Roles

Entra ID uses role-based administration — assign only the role needed.

### Common Admin Roles

| Role | What It Can Do |
|------|---------------|
| **Global Administrator** | Everything — no restrictions |
| **User Administrator** | Create/manage users and groups, reset passwords |
| **Password Administrator** | Reset passwords for non-admin users only |
| **Helpdesk Administrator** | Reset passwords, manage service requests |
| **Security Administrator** | Configure security features, view security reports |
| **Compliance Administrator** | Manage compliance features |
| **Exchange Administrator** | Manage Exchange Online |
| **SharePoint Administrator** | Manage SharePoint Online |
| **Teams Administrator** | Manage Microsoft Teams |
| **Intune Administrator** | Manage Intune device management |
| **Application Administrator** | Register and manage applications |
| **Billing Administrator** | Manage subscriptions and billing |

### GUI — Assign an Admin Role
1. Go to **Entra ID** → **Users** → click the user
2. Click **Assigned Roles** → **Add assignments**
3. Search for and select the role
4. Click **Add**

### PowerShell
```powershell
# Get all available roles
Get-AzureADDirectoryRoleTemplate | Select-Object DisplayName, Description

# Activate a role (if not already active)
Enable-AzureADDirectoryRole -RoleTemplateId (Get-AzureADDirectoryRoleTemplate |
  Where-Object DisplayName -eq "User Administrator").ObjectId

# Assign a role to a user
Add-AzureADDirectoryRoleMember `
  -ObjectId (Get-AzureADDirectoryRole | Where-Object DisplayName -eq "User Administrator").ObjectId `
  -RefObjectId (Get-AzureADUser -ObjectId "jsmith@company.com").ObjectId

# View members of a role
Get-AzureADDirectoryRoleMember `
  -ObjectId (Get-AzureADDirectoryRole | Where-Object DisplayName -eq "Global Administrator").ObjectId
```

---

## 5. Conditional Access Policies

Conditional Access is Entra ID's policy engine — it controls access based on conditions.

### Common Conditional Access Scenarios

| Policy | Condition | Action |
|--------|-----------|--------|
| Require MFA for admins | User is in admin role | Require MFA |
| Block legacy authentication | Auth method is legacy (SMTP, IMAP) | Block |
| Require compliant device | Accessing M365 apps | Require Intune-compliant device |
| Block risky sign-ins | Sign-in risk is high | Block or require MFA |
| Require MFA from untrusted locations | Location is outside corporate network | Require MFA |

### GUI — Create a Conditional Access Policy
1. Go to **Entra ID** → **Security** → **Conditional Access** → **New Policy**
2. Enter a **Name**
3. Configure **Users** — who the policy applies to (all users, specific groups, or exclude emergency accounts)
4. Configure **Target Resources** — which apps (All cloud apps, M365, Azure management, etc.)
5. Configure **Conditions**:
   - **Sign-in risk** — high, medium, low
   - **Device platforms** — Windows, iOS, Android
   - **Locations** — trusted networks, specific countries
   - **Client apps** — browser, mobile, legacy auth
6. Configure **Grant** controls:
   - **Require MFA**
   - **Require compliant device**
   - **Require Entra ID joined device**
7. Set **Policy state**: Report-only first, then On
8. Click **Create**

> ⚠️ **Warning:** Always exclude at least one **emergency access account** from Conditional Access policies to prevent lockout.

---

## 6. Multi-Factor Authentication

### MFA Methods Available
- **Microsoft Authenticator app** — push notification or TOTP code (recommended)
- **SMS text message** — one-time code via text
- **Phone call** — automated call
- **Hardware token** — FIDO2 security key or OATH token
- **Windows Hello** — biometric or PIN

### GUI — Configure MFA Settings
1. Go to **Entra ID** → **Security** → **MFA**
2. Click **Additional cloud-based MFA settings**
3. Configure:
   - Allowed methods for users
   - Remember MFA settings duration
   - Fraud alerts

### Per-User MFA (Legacy — use Conditional Access instead)
1. Go to **Entra ID** → **Users** → **Per-user MFA**
2. Find user → select → Enable or Enforce

---

## 7. Self-Service Password Reset (SSPR)

SSPR allows users to reset their own passwords without contacting IT.

### GUI — Enable SSPR
1. Go to **Entra ID** → **Password Reset**
2. Select scope:
   - **None** — SSPR disabled
   - **Selected** — specific groups only
   - **All** — all users
3. Configure **Authentication Methods**:
   - Number of methods required (1 or 2)
   - Available methods: email, phone, security questions, authenticator app
4. Configure **Registration** — require users to register at next sign-in
5. Click **Save**

---

## 8. Sign-In Logs and Audit Logs

### GUI — View Sign-In Logs
1. Go to **Entra ID** → **Monitoring** → **Sign-in logs**
2. Filter by:
   - User
   - Date range
   - Status (success, failure)
   - IP address
3. Click any sign-in for details including:
   - Location and IP
   - Device info
   - MFA result
   - Conditional Access result

### PowerShell
```powershell
# Get sign-in logs via Microsoft Graph
Connect-MgGraph -Scopes "AuditLog.Read.All"

# Get recent sign-ins
Get-MgAuditLogSignIn -Top 50 | Select-Object UserDisplayName, CreatedDateTime, Status

# Get failed sign-ins
Get-MgAuditLogSignIn -Filter "status/errorCode ne 0" -Top 50 |
  Select-Object UserDisplayName, CreatedDateTime, @{n="Error";e={$_.Status.FailureReason}}

# Get sign-ins from a specific user
Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'jsmith@company.com'" -Top 20

# Get audit logs (administrative changes)
Get-MgAuditLogDirectoryAudit -Top 50 |
  Select-Object ActivityDisplayName, ActivityDateTime, InitiatedBy
```

---

## 9. Hybrid Identity — Entra Connect

Hybrid identity synchronizes on-premises Active Directory users to Entra ID so users have one identity for both.

### Entra Connect Sync Modes

| Mode | Direction | Use Case |
|------|-----------|---------|
| **Password Hash Sync** | On-prem → Cloud | Simplest — passwords synced to cloud |
| **Pass-through Authentication** | On-prem validates | Password stays on-prem — no cloud hash |
| **Federation** | ADFS validates | Complex — used with ADFS |

### Basic Entra Connect Setup
```powershell
# Run on the Entra Connect server after installation

# Check sync status
Get-ADSyncScheduler

# Force immediate sync
Start-ADSyncSyncCycle -PolicyType Delta    # Sync changes only
Start-ADSyncSyncCycle -PolicyType Initial  # Full sync

# View sync errors
Get-ADSyncToolsSourceAnchorConflict

# Check connector status
Get-ADSyncConnector | Select-Object Name, ConnectorTypeName
```

---

## 10. Guest Users (B2B)

Guest users are external users invited to collaborate — they authenticate with their own organization's credentials.

### GUI — Invite a Guest User
1. Go to **Entra ID** → **Users** → **Invite external user**
2. Enter the external user's email address
3. Add a personal message (optional)
4. Assign them to a group if needed
5. Click **Invite**
6. The user receives an email and accepts the invitation

### PowerShell
```powershell
# Invite a guest user
New-AzureADMSInvitation `
  -InvitedUserEmailAddress "external@partner.com" `
  -InviteRedirectUrl "https://myapps.microsoft.com" `
  -SendInvitationMessage $true

# Get all guest users
Get-AzureADUser -Filter "userType eq 'Guest'" | Select-Object DisplayName, UserPrincipalName

# Remove a guest user
Remove-AzureADUser -ObjectId "external_partner.com#EXT#@company.onmicrosoft.com"
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| User can't sign in | Account disabled or locked | Check account status in Entra ID |
| MFA not working | Phone number wrong or app not configured | Reset MFA methods for user |
| Conditional Access blocking access | Policy too restrictive | Check CA policy in report-only mode first |
| Guest user can't access resources | Not added to correct groups | Add guest to appropriate M365 groups |
| Password sync not working | Entra Connect issue | Check sync status with `Get-ADSyncScheduler` |
| App SSO failing | App not properly configured | Check enterprise app configuration |
| Admin locked out | CA policy blocks admin | Use emergency access account to fix policy |

---

## Quick Reference

```powershell
# Connect
Connect-AzureAD

# Users
Get-AzureADUser -All $true                              # All users
Get-AzureADUser -ObjectId "user@domain.com"            # Specific user
New-AzureADUser -DisplayName "Name" -UserPrincipalName "user@domain.com"
Set-AzureADUser -ObjectId "user@domain.com" -AccountEnabled $false  # Disable
Remove-AzureADUser -ObjectId "user@domain.com"          # Delete

# Groups
Get-AzureADGroup -SearchString "GroupName"             # Find group
New-AzureADGroup -DisplayName "Group" -SecurityEnabled $true
Add-AzureADGroupMember -ObjectId $groupId -RefObjectId $userId

# Roles
Get-AzureADDirectoryRole                               # List active roles
Get-AzureADDirectoryRoleMember -ObjectId $roleId       # Role members

# Sign-in logs
Get-MgAuditLogSignIn -Top 50                           # Recent sign-ins
```

---

## Notes
- **Entra ID** is the new name for **Azure AD** — both names are still used in documentation and commands
- Always create at least two **emergency access accounts** (break-glass accounts) excluded from all Conditional Access — store passwords in a safe
- **Global Administrator** is extremely powerful — use PIM to require just-in-time activation instead of permanent assignment
- **Conditional Access** should be tested in **report-only mode** before enabling to avoid accidental lockouts
- Entra ID **sign-in logs** are only retained for **30 days** on free tier — export to Log Analytics for longer retention
- **Dynamic groups** automatically manage membership based on attributes — set up once and they self-maintain

---

## Related Documents
- [Microsoft 365 User Management](microsoft-365-user-management.md)
- [Intune Basics](intune-basics.md)
- [Azure Basics](azure-basics.md)
- [Active Directory User Management](../Windows/active-directory-user-management.md)
