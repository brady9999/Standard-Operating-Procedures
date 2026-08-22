# Intune Basics
> An introduction to Microsoft Intune — cloud-based endpoint management for devices and applications.

**Category:** Microsoft  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Intune** | Microsoft Intune | Microsoft's cloud-based MDM and MAM solution |
| **MDM** | Mobile Device Management | Managing entire devices — policies, apps, compliance |
| **MAM** | Mobile Application Management | Managing apps only — without full device control |
| **Enrollment** | Device Enrollment | The process of registering a device with Intune |
| **Compliance Policy** | Compliance Policy | Rules defining what makes a device compliant (e.g. encrypted, up to date) |
| **Configuration Profile** | Configuration Profile | Settings pushed to devices — Wi-Fi, VPN, restrictions |
| **App Protection Policy** | App Protection Policy | MAM policies controlling how apps handle corporate data |
| **Conditional Access** | Conditional Access | Entra ID policy requiring device compliance before granting access |
| **Autopilot** | Windows Autopilot | Zero-touch Windows device provisioning via Intune |
| **Company Portal** | Intune Company Portal | The app users install to enroll and access corporate resources |
| **Tenant** | Intune Tenant | Your organization's Intune environment |
| **Assignment** | Policy Assignment | Deploying a policy or app to users or devices |
| **Group** | Entra ID Group | Users or devices targeted by a policy |
| **Scope Tag** | Scope Tag | Labels to delegate admin access to specific devices |
| **BYOD** | Bring Your Own Device | Personal devices used for work |
| **Corporate-Owned** | Corporate-Owned Device | Company-issued and fully managed devices |
| **DEP/ADE** | Automated Device Enrollment | Apple's zero-touch enrollment for iOS/macOS |
| **Android Enterprise** | Android Enterprise | Google's managed Android platform for enterprise |

---

## Overview
Microsoft Intune is a cloud-based endpoint management solution that allows IT to:
- Enroll and manage Windows, macOS, iOS, and Android devices
- Push configuration settings and security policies
- Deploy applications remotely
- Enforce compliance requirements
- Wipe or lock lost/stolen devices

**Intune Admin Center:** https://intune.microsoft.com

**Intune integrates with:**
- **Entra ID** — for identity and Conditional Access
- **Microsoft 365** — for app deployment and data protection
- **Defender for Endpoint** — for threat protection
- **Azure** — for infrastructure management

---

## 1. Accessing the Intune Admin Center

1. Go to **https://intune.microsoft.com**
2. Sign in with an Intune administrator account
3. Key sections:
   - **Devices** — manage enrolled devices
   - **Apps** — deploy and manage applications
   - **Endpoint Security** — security policies and compliance
   - **Reports** — device and compliance reporting
   - **Tenant Administration** — connectors and settings

---

## 2. Device Enrollment

### Enrollment Methods by Platform

| Platform | Method | Use Case |
|----------|--------|---------|
| Windows | Autopilot | New corporate devices — zero touch |
| Windows | Entra ID Join | Corporate devices — user enrolls |
| Windows | MDM enrollment | Existing domain-joined devices |
| iOS/iPadOS | ADE (Apple DEP) | Corporate iOS devices — zero touch |
| iOS/iPadOS | Company Portal | BYOD or corporate devices |
| Android | Android Enterprise | Corporate Android devices |
| Android | Company Portal | BYOD devices |
| macOS | ADE | Corporate Macs — zero touch |
| macOS | Company Portal | Manual enrollment |

### Enrolling a Windows Device (User-Driven)

#### GUI — Step by Step on the Device
1. Open **Settings** → **Accounts** → **Access work or school**
2. Click **Connect**
3. Enter work email address
4. Sign in with M365 credentials
5. If prompted for MFA — complete it
6. Device enrolls and policies begin applying

### Checking Enrollment Status

#### Intune Admin Center
1. Go to **Devices** → **All Devices**
2. Find the device — view enrollment status, compliance, last check-in
3. Click a device for detailed info

### PowerShell — Check Enrollment Status
```powershell
# On the enrolled device — check MDM enrollment
dsregcmd /status

# Look for:
# AzureAdJoined: YES
# DomainJoined: NO (or YES for hybrid)
# MDMEnrolled: YES
```

---

## 3. Compliance Policies

Compliance policies define what makes a device "compliant" — meeting the minimum security requirements.

### GUI — Create a Compliance Policy
1. Go to **Devices** → **Compliance**
2. Click **Create Policy**
3. Select **Platform** (Windows 10 and later)
4. Click **Create**
5. Enter **Name** and **Description**
6. Configure settings:

**Common compliance settings for Windows:**

| Setting | Recommended Value |
|---------|------------------|
| Minimum OS version | 10.0.19041 (Windows 10 2004+) |
| BitLocker required | Yes |
| Secure Boot required | Yes |
| Code integrity required | Yes |
| Firewall required | Yes |
| Antivirus required | Yes |
| Antispyware required | Yes |
| Microsoft Defender Antimalware | Required |
| Password required | Yes |
| Minimum password length | 8 |
| Password complexity | Required |
| Maximum minutes of inactivity before screen lock | 15 |

7. Configure **Actions for noncompliance**:
   - Mark device as noncompliant immediately
   - Send email notification after X days
   - Retire device after X days

8. **Assignments** — assign to a group (e.g. All Devices or All Users)
9. Click **Create**

---

## 4. Configuration Profiles

Configuration profiles push settings to devices — Wi-Fi, VPN, email, restrictions, etc.

### GUI — Create a Configuration Profile
1. Go to **Devices** → **Configuration**
2. Click **Create** → **New Policy**
3. Select **Platform** and **Profile type**
4. Configure settings
5. Assign to groups
6. Click **Create**

### Common Profile Types

| Profile Type | What It Configures |
|-------------|-------------------|
| **Device Restrictions** | Camera, screen capture, browser settings, app store |
| **Wi-Fi** | Push Wi-Fi credentials so devices auto-connect |
| **VPN** | Configure VPN client settings |
| **Email** | Configure email account settings |
| **Certificate** | Deploy certificates for Wi-Fi/VPN authentication |
| **Endpoint Protection** | Windows Defender, firewall settings |
| **Administrative Templates** | Group Policy-style settings for Windows |
| **Custom** | Deploy custom OMA-URI settings |

### Windows Device Restriction Profile Example

```
Key settings to configure:
✅ Require BitLocker encryption
✅ Block screen capture
✅ Block camera on lock screen
✅ Require Windows Hello for Business
✅ Block unsigned apps
✅ Block removable storage (optional — strict environments)
✅ Configure Windows Update settings
```

---

## 5. Application Deployment

### GUI — Deploy an App
1. Go to **Apps** → **All Apps**
2. Click **Add**
3. Select app type:
   - **Microsoft 365 Apps** — Office suite
   - **Windows app (Win32)** — .exe or .msi installers
   - **Microsoft Store app** — from the Microsoft Store
   - **Web link** — add a website as an app shortcut
   - **Line-of-business app** — custom .msi/.intunewin
4. Configure app details
5. Set **Assignments**:
   - **Required** — automatically installs
   - **Available** — user can install via Company Portal
   - **Uninstall** — force removes the app

### Deploying Microsoft 365 Apps

1. Go to **Apps** → **Add** → **Microsoft 365 Apps** → **Windows 10 and later**
2. Configure:
   - Select apps to include (Word, Excel, Outlook, Teams, etc.)
   - Update channel (Current, Monthly Enterprise, Semi-Annual)
   - Architecture (64-bit recommended)
3. Assign to **All Users** or specific group as **Required**
4. Click **Create**

### Packaging a Win32 App

```powershell
# Download the Microsoft Win32 Content Prep Tool
# https://github.com/Microsoft/Microsoft-Win32-Content-Prep-Tool

# Convert .exe or .msi to .intunewin format
IntuneWinAppUtil.exe -c "C:\AppSource" -s "setup.exe" -o "C:\Output"

# Then upload the .intunewin file to Intune:
# Apps → Add → Windows app (Win32)
# Upload the .intunewin file
# Configure install/uninstall commands, detection rules
```

---

## 6. App Protection Policies (MAM)

App Protection Policies protect corporate data in apps without full device management — useful for BYOD.

### GUI — Create App Protection Policy
1. Go to **Apps** → **App Protection Policies**
2. Click **Create Policy** → select platform (iOS or Android)
3. Configure:

**Key settings:**
- **Data transfer** — allow/block copy/paste between managed and unmanaged apps
- **Access requirements** — require PIN to access apps
- **Conditional launch** — block jailbroken/rooted devices
- **Encryption** — encrypt data in managed apps

4. Assign to users (not devices — MAM is user-targeted)

---

## 7. Remote Actions

Intune allows you to perform remote actions on enrolled devices.

### GUI — Remote Actions
1. Go to **Devices** → **All Devices**
2. Click on a device
3. In the top menu you'll see available actions:

| Action | What It Does |
|--------|-------------|
| **Sync** | Force device to check in and apply latest policies |
| **Restart** | Remotely restart the device |
| **Lock** | Lock the device screen immediately |
| **Reset passcode** | Remove the PIN (iOS/Android) |
| **Wipe** | Factory reset — removes all data |
| **Retire** | Remove corporate data only (BYOD-friendly) |
| **Autopilot Reset** | Reinstall Windows while keeping Autopilot enrollment |
| **Collect diagnostics** | Pull logs from the device for troubleshooting |

### PowerShell — Remote Actions via Graph API
```powershell
# Install Microsoft Graph module
Install-Module Microsoft.Graph -Force

# Connect
Connect-MgGraph -Scopes "DeviceManagementManagedDevices.ReadWrite.All"

# Get all managed devices
Get-MgDeviceManagementManagedDevice | Select-Object DeviceName, OperatingSystem, ComplianceState

# Remote lock a device
$deviceId = "device-object-id"
Invoke-MgDeviceManagementManagedDeviceRemoteLock -ManagedDeviceId $deviceId

# Sync a device
Invoke-MgDeviceManagementManagedDeviceSyncDevice -ManagedDeviceId $deviceId

# Wipe a device
Invoke-MgDeviceManagementManagedDeviceWipe -ManagedDeviceId $deviceId
```

---

## 8. Windows Autopilot

Autopilot enables zero-touch Windows deployment — devices ship directly to users and self-configure.

### Autopilot Process
```
1. IT registers device hardware hash with Autopilot (via CSV or OEM)
2. Device ships to user
3. User powers on → connects to internet
4. Device contacts Microsoft → recognized as Autopilot device
5. User signs in with work account
6. Windows automatically configures, joins Entra ID, enrolls in Intune
7. Policies and apps deploy automatically
8. User is ready to work — no IT touch required
```

### Registering a Device for Autopilot
```powershell
# Install module
Install-Module WindowsAutoPilotIntune -Force

# Get hardware hash from the device
$hash = Get-WindowsAutoPilotInfo
$hash | Out-File "autopilot-hash.csv"

# Upload CSV to Intune:
# Devices → Enrollment → Windows → Windows Autopilot Devices → Import
```

---

## 9. Monitoring and Reporting

### GUI — Device Compliance Report
1. Go to **Reports** → **Device Compliance**
2. View compliance state across all devices
3. Filter by platform, compliance state, OS version

### GUI — Device Status for a Policy
1. Go to **Devices** → **Configuration** (or Compliance)
2. Click a policy
3. Click **Device and user check-in status**
4. Shows which devices have received and applied the policy

### PowerShell — Compliance Status
```powershell
# Get all noncompliant devices
Get-MgDeviceManagementManagedDevice |
  Where-Object ComplianceState -eq "noncompliant" |
  Select-Object DeviceName, UserDisplayName, ComplianceState, LastSyncDateTime
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Device shows as noncompliant | Compliance policy requirements not met | Check device for BitLocker, OS version, password |
| Policy not applying | Device not checked in | Force sync: Devices → select device → Sync |
| App not installing | Assignment not correct or app error | Check app deployment status in Intune |
| Enrollment fails | License not assigned or wrong MDM authority | Assign Intune license, check MDM authority |
| BitLocker not enforcing | TPM not present or disabled | Check BIOS for TPM, enable Secure Boot |
| Device shows duplicate entries | Re-enrolled without removing old entry | Delete old device entry in Intune |

---

## Quick Reference

```
Admin Center: https://intune.microsoft.com

Key sections:
- Devices → All Devices        View and manage enrolled devices
- Devices → Compliance         Create and assign compliance policies
- Devices → Configuration      Create and assign configuration profiles
- Apps → All Apps              Deploy and manage applications
- Apps → App Protection        MAM policies for BYOD
- Reports                      Compliance and enrollment reports

Remote actions (on a device):
Sync → Lock → Wipe → Retire → Restart → Collect diagnostics
```

---

## Notes
- **Compliance policies** work with **Conditional Access** — noncompliant devices can be blocked from M365
- **Retire** removes corporate data and unenrolls the device but leaves personal data — use for BYOD
- **Wipe** factory resets the device — use for lost/stolen corporate devices
- Intune requires an **Intune license** — included in M365 E3, E5, and EMS plans
- Policy changes can take up to **8 hours** to apply — use **Sync** to force immediate check-in
- **Autopilot** dramatically reduces IT workload for new device setup — worth implementing from day one

---

## Related Documents
- [Entra ID Basics](entra-id-basics.md)
- [Microsoft 365 User Management](microsoft-365-user-management.md)
- [Azure Basics](azure-basics.md)
