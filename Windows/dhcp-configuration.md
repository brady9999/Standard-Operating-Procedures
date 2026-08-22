# DHCP Configuration
> A guide to installing, configuring, and managing DHCP on Windows Server.

**Category:** Windows  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **DHCP** | Dynamic Host Configuration Protocol | Automatically assigns IP addresses and network settings to devices |
| **Scope** | DHCP Scope | A range of IP addresses the DHCP server can assign |
| **Lease** | IP Lease | The amount of time a device is allowed to keep an assigned IP address |
| **Reservation** | DHCP Reservation | A permanent IP assignment tied to a specific device's MAC address |
| **Exclusion** | Scope Exclusion | An IP address or range within the scope that will never be assigned |
| **Option** | DHCP Option | Additional network settings sent with the IP — e.g. DNS server, gateway |
| **Scope Option** | Scope Option | Options applied to a specific scope |
| **Server Option** | Server Option | Options applied to all scopes on the server |
| **MAC Address** | Media Access Control Address | The unique hardware identifier of a network adapter |
| **DORA** | Discover, Offer, Request, Acknowledge | The four steps of the DHCP lease process |
| **Superscope** | Superscope | A group of scopes managed together |
| **Failover** | DHCP Failover | A configuration where two DHCP servers share responsibility for a scope |
| **Authorized** | DHCP Authorization | The process of registering a DHCP server with Active Directory |

---

## Overview
DHCP automatically assigns IP addresses, subnet masks, default gateways, and DNS server addresses to devices when they connect to the network. Without DHCP every device would need to be manually configured with a static IP.

**The DORA Process — How DHCP Works:**
1. **Discover** — Client broadcasts "I need an IP address"
2. **Offer** — DHCP server responds "I can give you 192.168.1.50"
3. **Request** — Client replies "Yes please, I'll take 192.168.1.50"
4. **Acknowledge** — Server confirms "192.168.1.50 is yours for 8 hours"

---

## Prerequisites
- Windows Server with DHCP Server role installed
- Static IP address on the DHCP server
- Administrator rights
- DHCP server must be authorized in Active Directory

---

## How to Open DHCP Manager

**Method 1 — Server Manager:**
1. Open **Server Manager**
2. Click **Tools** → **DHCP**

**Method 2 — Run Dialog:**
1. Press **Windows + R**
2. Type `dhcpmgmt.msc`
3. Press **Enter**

---

## 1. Installing the DHCP Server Role

### GUI — Step by Step
1. Open **Server Manager**
2. Click **Manage** → **Add Roles and Features**
3. Click **Next** until **Server Roles**
4. Check **DHCP Server** → **Add Features**
5. Click **Next** through remaining pages → **Install**
6. After installation click **Complete DHCP Configuration**
7. Click **Commit** to authorize the server in Active Directory
8. Click **Close**

### PowerShell
```powershell
# Install DHCP role
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Authorize the DHCP server in Active Directory
Add-DhcpServerInDC -DnsName "server.company.com" -IPAddress 192.168.1.10

# Verify authorization
Get-DhcpServerInDC
```

---

## 2. Creating a DHCP Scope

A scope defines the pool of IP addresses the server can hand out.

### GUI — Step by Step
1. Open **DHCP Manager**
2. Expand your server name
3. Right-click **IPv4** → **New Scope**
4. The **New Scope Wizard** opens → click **Next**
5. Enter a **Name** and **Description** e.g. `Office LAN` → **Next**
6. Enter the IP address range:
   - **Start IP address** e.g. `192.168.1.1`
   - **End IP address** e.g. `192.168.1.254`
   - **Length** or **Subnet mask** e.g. `255.255.255.0` → **Next**
7. Add any **exclusions** (IPs that should never be assigned) → **Next**
8. Set the **Lease Duration** (default 8 days) → **Next**
9. Choose **Yes, I want to configure these options now** → **Next**
10. Enter the **Default Gateway** (router IP) e.g. `192.168.1.1` → **Add** → **Next**
11. Enter **DNS Server** IP addresses → **Next**
12. Enter **WINS Server** if applicable → **Next**
13. Choose **Yes, I want to activate this scope now** → **Next** → **Finish**

### PowerShell
```powershell
# Create a new scope
Add-DhcpServerv4Scope -Name "Office LAN" `
  -StartRange 192.168.1.100 `
  -EndRange 192.168.1.200 `
  -SubnetMask 255.255.255.0 `
  -Description "Main office network" `
  -State Active

# Set scope options (gateway, DNS)
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 `
  -Router 192.168.1.1 `
  -DnsServer 192.168.1.10 `
  -DnsDomain "company.com"
```

---

## 3. Adding Exclusions to a Scope

Exclusions prevent certain IPs from being assigned — useful for static devices like servers, printers, and network equipment.

### GUI — Step by Step
1. Open **DHCP Manager**
2. Expand **IPv4** → expand your scope → click **Address Pool**
3. Right-click **Address Pool** → **New Exclusion Range**
4. Enter:
   - **Start IP address** e.g. `192.168.1.1`
   - **End IP address** e.g. `192.168.1.20`
5. Click **Add** → **Close**

### PowerShell
```powershell
# Add an exclusion range
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 `
  -StartRange 192.168.1.1 `
  -EndRange 192.168.1.20

# View exclusions
Get-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0
```

---

## 4. Creating a Reservation

A reservation permanently assigns a specific IP to a device based on its MAC address. The device always gets the same IP even though DHCP is assigning it.

### GUI — Step by Step
1. Open **DHCP Manager**
2. Expand your scope → click **Reservations**
3. Right-click **Reservations** → **New Reservation**
4. Enter:
   - **Reservation name** e.g. `Office Printer`
   - **IP address** e.g. `192.168.1.50` (must be within scope range)
   - **MAC address** e.g. `00-1A-2B-3C-4D-5E` (find on device label or via `ipconfig /all`)
   - **Description** — optional
5. Select **Both** for supported types
6. Click **Add** → **Close**

### PowerShell
```powershell
# Create a reservation
Add-DhcpServerv4Reservation -ScopeId 192.168.1.0 `
  -IPAddress 192.168.1.50 `
  -ClientId "00-1A-2B-3C-4D-5E" `
  -Name "Office Printer" `
  -Description "HP LaserJet in main office"

# View all reservations
Get-DhcpServerv4Reservation -ScopeId 192.168.1.0
```

---

## 5. Viewing Active Leases

### GUI — Step by Step
1. Open **DHCP Manager**
2. Expand your scope
3. Click **Address Leases**
4. All current leases appear in the right panel showing:
   - IP address
   - Client name
   - Lease expiration
   - MAC address

### PowerShell
```powershell
# View all active leases
Get-DhcpServerv4Lease -ScopeId 192.168.1.0

# Find a specific device by IP
Get-DhcpServerv4Lease -ScopeId 192.168.1.0 -IPAddress 192.168.1.50

# Find leases by hostname
Get-DhcpServerv4Lease -ScopeId 192.168.1.0 | Where-Object {$_.HostName -like "*printer*"}
```

---

## 6. Configuring DHCP Options

DHCP options are additional settings sent to clients along with the IP address.

| Option Number | Option Name | What It Does |
|--------------|-------------|--------------|
| **003** | Router | Default gateway |
| **006** | DNS Servers | DNS server addresses |
| **015** | DNS Domain Name | Domain suffix e.g. `company.com` |
| **044** | WINS Servers | WINS server addresses |
| **066** | Boot Server | PXE boot server |

### GUI — Step by Step
1. Open **DHCP Manager**
2. Expand your scope → right-click **Scope Options** → **Configure Options**
3. Check the option you want to configure
4. Enter the value
5. Click **Apply** → **OK**

### PowerShell
```powershell
# Set DNS server and domain for a scope
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 `
  -DnsServer 192.168.1.10, 192.168.1.11 `
  -DnsDomain "company.com"

# Set default gateway
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -Router 192.168.1.1

# View current options
Get-DhcpServerv4OptionValue -ScopeId 192.168.1.0
```

---

## 7. Activating and Deactivating a Scope

### GUI — Step by Step
1. Right-click the scope
2. Select **Activate** or **Deactivate**
3. Confirm if prompted

### PowerShell
```powershell
# Activate
Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -State Active

# Deactivate
Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -State InActive
```

---

## 8. Authorizing the DHCP Server

In an Active Directory environment DHCP servers must be authorized to prevent rogue DHCP servers from handing out wrong addresses.

### GUI — Step by Step
1. Open **DHCP Manager**
2. Right-click your server name
3. Select **Authorize**
4. The server icon changes from red to green

### PowerShell
```powershell
# Authorize server
Add-DhcpServerInDC -DnsName "server.company.com" -IPAddress 192.168.1.10

# Check authorization status
Get-DhcpServerInDC
```

---

## Finding a Device's MAC Address

```powershell
# On the device itself
ipconfig /all
# Look for "Physical Address"

# Remotely via PowerShell
Get-WmiObject Win32_NetworkAdapterConfiguration -ComputerName "PC-NAME" |
  Where-Object {$_.IPEnabled} | Select-Object MACAddress

# From the DHCP server by IP
Get-DhcpServerv4Lease -ScopeId 192.168.1.0 -IPAddress 192.168.1.50 |
  Select-Object ClientId
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Devices getting 169.254.x.x addresses | Not reaching DHCP server | Check DHCP server is running, authorized, and scope is active |
| IP conflict on network | Static IP conflicts with DHCP range | Add static IP to exclusion range or move to outside scope range |
| DHCP server not responding | Service stopped | Run `Get-Service DHCPServer` and restart if needed |
| Scope running out of IPs | Scope too small or leases too long | Expand scope range or reduce lease duration |
| Wrong DNS being assigned | Scope options misconfigured | Check scope options — DNS server addresses |
| Can't authorize server | Not a Domain Admin | Log in as Domain Admin and authorize |

---

## Quick Reference

```powershell
# View all scopes
Get-DhcpServerv4Scope

# View all leases in a scope
Get-DhcpServerv4Lease -ScopeId 192.168.1.0

# View all reservations
Get-DhcpServerv4Reservation -ScopeId 192.168.1.0

# View scope options
Get-DhcpServerv4OptionValue -ScopeId 192.168.1.0

# Restart DHCP service
Restart-Service DHCPServer

# Check DHCP service status
Get-Service DHCPServer

# View server statistics
Get-DhcpServerv4Statistics
```

---

## Notes
- Never run two unauthorized DHCP servers on the same network — they will conflict and cause IP address issues
- Always exclude the IP range used by static devices before activating a scope
- Lease duration of 8 days is standard for most office environments
- For guest networks use shorter leases (4-8 hours) since devices change frequently
- DHCP and DNS can be configured to automatically update DNS records when a lease is assigned — called DDNS (Dynamic DNS)

---

## Related Documents
- [DNS Configuration](dns-configuration.md)
- [Active Directory Domain Configuration](active-directory-domain-configuration.md)
- [VLAN Configuration](../Networking/vlan-configuration-cisco-switch.md)
