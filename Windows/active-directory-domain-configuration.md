# Active Directory Domain Configuration
> A guide to installing and configuring Active Directory Domain Services and promoting a Windows Server to a Domain Controller.

**Category:** Windows  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **AD DS** | Active Directory Domain Services | The Windows Server role that provides directory services |
| **DC** | Domain Controller | A server that runs AD DS and handles authentication |
| **Forest** | AD Forest | The top-level container — holds one or more domains |
| **Domain** | AD Domain | A logical grouping of users, computers, and resources |
| **Tree** | Domain Tree | A collection of domains that share a common DNS namespace |
| **FQDN** | Fully Qualified Domain Name | The complete domain name e.g. `company.com` |
| **NetBIOS** | NetBIOS Name | The short name of the domain e.g. `COMPANY` |
| **FSMO** | Flexible Single Master Operations | Special roles held by specific DCs in the domain |
| **GC** | Global Catalog | A DC that holds a partial copy of all objects in the forest |
| **SYSVOL** | System Volume | A shared folder on DCs that stores group policy and logon scripts |
| **NTDS** | NT Directory Services | The AD database file `ntds.dit` stored on the DC |
| **PDC** | Primary Domain Controller | The FSMO role that handles password changes and time synchronization |
| **Functional Level** | Domain/Forest Functional Level | Determines which AD features are available based on DC versions |

---

## Overview
Active Directory Domain Services (AD DS) is the heart of a Windows enterprise network. When you promote a server to a Domain Controller it becomes the authority for:
- Authenticating users logging into domain-joined computers
- Storing all user, computer, and group objects
- Applying Group Policy to users and computers
- Providing DNS for the domain

Every Windows enterprise network needs at least one Domain Controller. Best practice is to have at least two for redundancy.

---

## Prerequisites
- Windows Server installed with a static IP address
- DNS Server role installed (or will be installed alongside AD DS)
- The server should NOT be renamed after promotion — name it correctly first
- Administrator rights
- Minimum 4GB RAM, 40GB disk space recommended

> ⚠️ **Important:** Set a static IP address BEFORE promoting to Domain Controller. A DC with a dynamic IP will cause authentication and DNS failures.

---

## Setting a Static IP Before Promotion

### GUI — Step by Step
1. Right-click the network icon in the taskbar → **Open Network & Internet Settings**
2. Click **Change adapter options**
3. Right-click your network adapter → **Properties**
4. Select **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**
5. Select **Use the following IP address** and enter:
   - **IP address** e.g. `192.168.1.10`
   - **Subnet mask** e.g. `255.255.255.0`
   - **Default gateway** e.g. `192.168.1.1`
6. Under DNS — set **Preferred DNS server** to `127.0.0.1` (itself, since it will be the DNS server)
7. Click **OK** → **Close**

### PowerShell
```powershell
# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" `
  -IPAddress 192.168.1.10 `
  -PrefixLength 24 `
  -DefaultGateway 192.168.1.1

# Set DNS to itself
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1
```

---

## 1. Installing the AD DS Role

### GUI — Step by Step
1. Open **Server Manager**
2. Click **Manage** → **Add Roles and Features**
3. Click **Next** until **Server Roles**
4. Check **Active Directory Domain Services**
5. Click **Add Features** when prompted
6. Click **Next** through remaining pages → **Install**
7. Wait for installation → **Close**
8. In Server Manager a yellow warning flag appears — click it
9. Click **Promote this server to a domain controller**

### PowerShell
```powershell
# Install AD DS role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

---

## 2. Promoting the Server to a Domain Controller

### GUI — New Forest (First DC in the environment)
1. After clicking **Promote this server to a domain controller**
2. Select **Add a new forest**
3. Enter the **Root domain name** e.g. `company.com` → **Next**
4. Set **Forest functional level** and **Domain functional level** — choose the highest your environment supports
5. Check **Domain Name System (DNS) server** — leave checked
6. Check **Global Catalog (GC)** — leave checked
7. Enter a **DSRM password** (Directory Services Restore Mode — used for AD recovery) → **Next**
8. DNS delegation warning — click **Next**
9. Verify the **NetBIOS domain name** e.g. `COMPANY` → **Next**
10. Review **Paths** for NTDS, SYSVOL, and log files — defaults are fine → **Next**
11. Review all settings → **Next**
12. Prerequisites check runs — if all pass click **Install**
13. Server will automatically restart after promotion

### GUI — Additional DC (Joining an existing domain)
1. Select **Add a domain controller to an existing domain**
2. Enter the domain name e.g. `company.com`
3. Enter credentials of a Domain Admin → **Next**
4. Check **DNS server** and **Global Catalog** → **Next**
5. Enter DSRM password → **Next**
6. Complete the wizard → **Install**

### PowerShell — New Forest
```powershell
# Install AD DS and promote to DC in one command
Install-ADDSForest `
  -DomainName "company.com" `
  -DomainNetbiosName "COMPANY" `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "DSRMPass123!" -AsPlainText -Force) `
  -Force
```

### PowerShell — Additional DC
```powershell
Install-ADDSDomainController `
  -DomainName "company.com" `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "DSRMPass123!" -AsPlainText -Force) `
  -Credential (Get-Credential) `
  -Force
```

---

## 3. Joining a Computer to the Domain

After the DC is set up you need to join client computers to the domain.

### GUI — Windows 10/11 — Step by Step
1. Right-click **Start** → **System**
2. Click **Rename this PC (advanced)**
3. Click the **Change** button
4. Select **Domain** and enter the domain name e.g. `company.com`
5. Click **OK**
6. Enter Domain Admin credentials when prompted
7. Click **OK** — you will see a welcome message
8. Click **OK** → **Close** → **Restart Now**

### PowerShell
```powershell
# Join computer to domain
Add-Computer -DomainName "company.com" `
  -Credential (Get-Credential) `
  -Restart
```

---

## 4. Verifying the Domain Controller

### GUI — Step by Step
1. Open **Active Directory Users and Computers** (`dsa.msc`)
2. You should see your domain listed with default OUs
3. Open **DNS Manager** (`dnsmgmt.msc`) — you should see the domain zone with AD records
4. Open **Active Directory Sites and Services** — verify the DC appears

### PowerShell
```powershell
# Verify domain info
Get-ADDomain

# Check DC list
Get-ADDomainController -Filter *

# Verify FSMO roles
netdom query fsmo

# Test AD replication (if multiple DCs)
repadmin /showrepl

# Check domain and forest functional levels
Get-ADDomain | Select-Object DomainMode
Get-ADForest | Select-Object ForestMode
```

---

## 5. FSMO Roles

Every AD domain has five FSMO roles. By default the first DC holds all five.

| Role | Scope | Purpose |
|------|-------|---------|
| **Schema Master** | Forest | Controls AD schema changes |
| **Domain Naming Master** | Forest | Controls adding/removing domains |
| **PDC Emulator** | Domain | Password changes, time sync, Group Policy |
| **RID Master** | Domain | Allocates unique IDs to objects |
| **Infrastructure Master** | Domain | Manages cross-domain references |

### PowerShell
```powershell
# View FSMO role holders
netdom query fsmo

# Or via PowerShell
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster
```

---

## 6. Configuring Sites and Subnets

Sites help AD route authentication requests efficiently, especially in multi-location environments.

### GUI — Step by Step
1. Open **Active Directory Sites and Services** (`dssite.msc`)
2. Expand **Sites** — you will see **Default-First-Site-Name**
3. Right-click **Default-First-Site-Name** → **Rename** — give it a meaningful name e.g. `Winnipeg-HQ`
4. To add a subnet:
   - Right-click **Subnets** → **New Subnet**
   - Enter the network e.g. `192.168.1.0/24`
   - Associate it with the correct site
   - Click **OK**

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Promotion fails — DNS error | DNS not installed or misconfigured | Install DNS role first, or let AD DS install it during promotion |
| Can't join computer to domain | DNS pointing to wrong server | Set client DNS to point to the DC's IP |
| Authentication failures after promotion | Time skew between DC and clients | Sync time — PDC Emulator is the time authority |
| AD replication failing | Network issue or DNS problem | Run `repadmin /showrepl` to diagnose |
| Can't find DC | DNS records missing | Check DNS for `_msdcs` records, run `dcdiag` |

---

## Useful Diagnostic Commands

```powershell
# Run a full DC diagnostic
dcdiag

# Check AD replication
repadmin /showrepl
repadmin /replsummary

# Test domain connectivity from a client
nltest /dsgetdc:company.com

# Check SYSVOL and NETLOGON shares
net share

# View all DCs in the domain
Get-ADDomainController -Filter *

# Force AD replication
repadmin /syncall /AdeP
```

---

## Notes
- Always have at least two Domain Controllers for redundancy — if the only DC goes down nobody can log in
- The DSRM password is critical — store it securely. It is used to recover AD if the DC has issues
- Never rename or change the IP of a DC without careful planning
- The PDC Emulator is the most important FSMO role for day-to-day operations — if it goes down password changes and Group Policy stop working
- Regularly back up the AD database (`ntds.dit`) and System State

---

## Related Documents
- [Active Directory User Management](active-directory-user-management.md)
- [Active Directory Password & Group Policy](active-directory-password-local--group-policy.md)
- [DNS Configuration](dns-configuration.md)
- [DHCP Configuration](dhcp-configuration.md)
