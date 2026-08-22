# Remote Desktop Configuration
> A guide to enabling, configuring, and connecting to Windows machines via Remote Desktop Protocol (RDP).

**Category:** Windows  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **RDP** | Remote Desktop Protocol | Microsoft's protocol for connecting to and controlling a remote Windows machine |
| **RDS** | Remote Desktop Services | The Windows Server role that enables multiple users to connect remotely |
| **mstsc** | Microsoft Terminal Services Client | The command to launch the Remote Desktop Connection client |
| **NLA** | Network Level Authentication | A security feature requiring authentication before the remote session is established |
| **Gateway** | RD Gateway | A server that allows RDP connections over HTTPS from outside the network |
| **CAL** | Client Access License | A license required for each user or device connecting to RDS |
| **Session** | Remote Session | An active connection between a client and a remote machine |
| **Shadowing** | Session Shadowing | Viewing or controlling another user's active remote session |

---

## Overview
Remote Desktop Protocol (RDP) allows you to connect to and control a Windows machine over the network as if you were sitting in front of it. It is one of the most commonly used tools in IT support for:
- Supporting users remotely without being physically present
- Managing servers without needing a monitor connected
- Accessing your work computer from home

RDP runs on **port 3389** by default.

---

## Prerequisites
- The remote machine must be running Windows Pro, Enterprise, or Server (Home edition does not support incoming RDP)
- The connecting machine can run any edition of Windows, macOS, iOS, or Android
- Both machines must be on the same network, or a VPN must be configured
- Administrator rights on the remote machine to enable RDP
- Firewall must allow port 3389

---

## 1. Enabling Remote Desktop on the Host Machine

### GUI — Windows 10/11 — Step by Step
1. Click **Start** → **Settings** (gear icon)
2. Go to **System** → **Remote Desktop**
3. Toggle **Enable Remote Desktop** to **On**
4. A confirmation dialog appears — click **Confirm**
5. Note the **PC name** shown on this page — you will need it to connect
6. Optional — click **Advanced settings** to:
   - Require Network Level Authentication (recommended)
   - Change the RDP port

### GUI — Windows Server — Step by Step
1. Open **Server Manager**
2. Click **Local Server** in the left panel
3. Find **Remote Desktop** — click **Disabled**
4. The System Properties window opens to the **Remote** tab
5. Select **Allow remote connections to this computer**
6. Check **Allow connections only from computers running Remote Desktop with Network Level Authentication** (recommended)
7. Click **Select Users** to add specific users who can connect
8. Click **Apply** → **OK**

### PowerShell
```powershell
# Enable RDP
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' `
  -Name "fDenyTSConnections" -Value 0

# Enable NLA
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' `
  -Name "UserAuthentication" -Value 1

# Allow RDP through Windows Firewall
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

# Confirm RDP is enabled
Get-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' `
  -Name "fDenyTSConnections"
# Value 0 = enabled, Value 1 = disabled
```

---

## 2. Adding Users Who Can Connect via RDP

By default only administrators can connect via RDP. To allow standard users:

### GUI — Step by Step
1. Go to **Settings** → **System** → **Remote Desktop**
2. Click **Select users that can remotely access this PC**
3. Click **Add**
4. Type the username → **Check Names** → **OK**
5. Click **OK**

### GUI — Via System Properties
1. Press **Windows + R** → type `sysdm.cpl` → Enter
2. Click the **Remote** tab
3. Click **Select Users**
4. Click **Add** → type username → **Check Names** → **OK**

### PowerShell
```powershell
# Add a user to the Remote Desktop Users group
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "username"

# View current members of the group
Get-LocalGroupMember -Group "Remote Desktop Users"
```

---

## 3. Connecting to a Remote Machine

### GUI — Step by Step
1. Press **Windows + R** → type `mstsc` → Enter
2. The **Remote Desktop Connection** window opens
3. In the **Computer** field enter:
   - Computer name e.g. `DESKTOP-ABC123`
   - IP address e.g. `192.168.1.50`
   - Domain and name e.g. `company.com\PC-NAME`
4. Click **Show Options** to configure:
   - **Username** — pre-fill the username
   - **Display** — set resolution and color depth
   - **Local Resources** — share local drives, printers, clipboard
   - **Experience** — connection speed optimization
5. Click **Connect**
6. Enter the password when prompted
7. If a certificate warning appears — click **Yes** to proceed (or **Don't ask me again** for trusted machines)

### PowerShell / CMD
```powershell
# Launch RDP connection directly
mstsc /v:192.168.1.50

# Connect with full screen
mstsc /v:192.168.1.50 /f

# Connect with specific resolution
mstsc /v:192.168.1.50 /w:1920 /h:1080

# Connect using saved .rdp file
mstsc "C:\connections\server01.rdp"
```

---

## 4. Saving an RDP Connection

### GUI — Step by Step
1. Open **Remote Desktop Connection** (`mstsc`)
2. Enter the computer name and username
3. Click **Show Options**
4. Configure all settings as needed
5. Click **Save As** at the bottom
6. Choose a location and filename e.g. `server01.rdp`
7. Click **Save**
8. Next time double-click the `.rdp` file to connect instantly

---

## 5. Changing the RDP Port

The default RDP port is 3389. Changing it adds a layer of security by obscuring the service from automated scanners.

> ⚠️ **Warning:** If you change the port you must specify it when connecting e.g. `192.168.1.50:3390`

### PowerShell
```powershell
# Change RDP port to 3390 (example)
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' `
  -Name "PortNumber" -Value 3390

# Update firewall rule for new port
New-NetFirewallRule -DisplayName "RDP Custom Port" `
  -Direction Inbound -Protocol TCP -LocalPort 3390 -Action Allow

# Restart RDP service to apply
Restart-Service -Name "TermService"
```

---

## 6. Disconnecting vs Logging Off

| Action | What Happens |
|--------|--------------|
| **Disconnect** (X button) | Session stays active in the background — programs keep running |
| **Log Off** | Session ends completely — all programs close |
| **Lock** | Screen locks but session stays active |

> **Best Practice:** Always **Log Off** when finished on a shared server to free up resources. **Disconnect** is acceptable on your own dedicated machine.

### To Log Off via PowerShell
```powershell
logoff
```

---

## 7. Viewing and Managing Active RDP Sessions (Server)

### GUI — Step by Step
1. Open **Task Manager** on the server
2. Click the **Users** tab
3. You can see all active and disconnected sessions
4. Right-click a session to:
   - **Send Message** — send a pop-up message to the user
   - **Disconnect** — disconnect their session
   - **Sign Off** — force log off

### PowerShell
```powershell
# List all active sessions
query session

# Or
qwinsta

# Disconnect a specific session by ID
logoff [session-id]

# Example — disconnect session 2
logoff 2
```

---

## 8. Enabling RDP Remotely via PowerShell

Useful when you need to enable RDP on a machine you can't physically access but have admin rights via PowerShell remoting.

```powershell
# Enable RDP on a remote machine
Invoke-Command -ComputerName "PC-NAME" -ScriptBlock {
  Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' `
    -Name "fDenyTSConnections" -Value 0
  Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
}
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Can't connect — connection refused | RDP not enabled or firewall blocking port 3389 | Enable RDP in settings and check firewall rules |
| Can't connect — credentials rejected | Wrong username/password or user not in RDP group | Verify credentials and add user to Remote Desktop Users group |
| Black screen after connecting | Display driver issue or slow connection | Try reducing color depth in Display settings |
| Session disconnects frequently | Network instability or idle timeout policy | Check network stability and adjust Group Policy timeout settings |
| Certificate warning every time | Self-signed certificate | Add machine to Trusted Computers or install a proper certificate |
| Only one user can connect | Windows 10/11 limits concurrent RDP sessions | Use Windows Server with RDS role for multiple simultaneous connections |
| RDP slow or laggy | Connection speed or visual settings | In RDP options set Experience to Low Speed Broadband or Modem |

---

## Quick Reference

```powershell
# Enable RDP
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0

# Disable RDP
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 1

# Check if RDP is enabled (0 = enabled, 1 = disabled)
Get-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections"

# Allow RDP through firewall
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

# List active sessions
query session

# Connect via command line
mstsc /v:COMPUTER-NAME-OR-IP
```

---

## Notes
- RDP on port 3389 is a common attack target — never expose it directly to the internet without a VPN or RD Gateway
- Always use Network Level Authentication (NLA) for added security
- Windows Home edition cannot accept incoming RDP connections but can initiate them
- macOS users can connect using **Microsoft Remote Desktop** from the App Store
- Mobile users can use the **Microsoft Remote Desktop** app on iOS and Android

---

## Related Documents
- [VPN Installation](../Networking/vpn-installation.md)
- [Active Directory User Management](active-directory-user-management.md)
- [Windows Event Viewer](windows-event-viewer.md)
