# DNS Configuration
> A guide to installing, configuring, and managing DNS on Windows Server.

**Category:** Windows  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **DNS** | Domain Name System | Translates human-readable domain names into IP addresses |
| **DNS Server** | DNS Server | A server that responds to DNS queries |
| **Zone** | DNS Zone | A portion of the DNS namespace managed by a specific DNS server |
| **Forward Lookup Zone** | Forward Lookup Zone | Resolves domain names to IP addresses (name → IP) |
| **Reverse Lookup Zone** | Reverse Lookup Zone | Resolves IP addresses to domain names (IP → name) |
| **A Record** | Address Record | Maps a hostname to an IPv4 address |
| **AAAA Record** | IPv6 Address Record | Maps a hostname to an IPv6 address |
| **CNAME** | Canonical Name Record | Creates an alias pointing to another hostname |
| **MX Record** | Mail Exchange Record | Specifies the mail server for a domain |
| **PTR Record** | Pointer Record | Used in reverse lookup zones to map IP to hostname |
| **SOA** | Start of Authority | Contains information about the DNS zone |
| **NS Record** | Name Server Record | Identifies the authoritative DNS servers for a domain |
| **TTL** | Time To Live | How long a DNS record is cached before being refreshed |
| **Forwarder** | DNS Forwarder | An external DNS server queries are sent to when the local server can't resolve them |
| **Root Hints** | Root Hints | A list of root DNS servers used as a last resort for resolving names |
| **Recursion** | DNS Recursion | The DNS server queries other servers on behalf of the client |

---

## Overview
DNS is the phonebook of the internet and your network. When you type `google.com` your computer asks a DNS server "what is the IP address for google.com?" and the DNS server responds with the answer so your computer knows where to send the request.

In a Windows Server environment DNS is tightly integrated with Active Directory. Every domain-joined computer uses the DNS server to locate domain controllers, services, and other computers on the network.

**How a DNS query works:**
1. User types `google.com` in browser
2. Computer checks its local DNS cache
3. If not cached — queries the configured DNS server
4. DNS server checks its own records
5. If not found — forwards the query to an upstream server (forwarder or root hints)
6. Answer is returned and cached for the TTL duration

---

## Prerequisites
- Windows Server with DNS Server role installed
- Static IP address on the DNS server
- Administrator rights

---

## How to Open DNS Manager

**Method 1 — Server Manager:**
1. Open **Server Manager**
2. Click **Tools** → **DNS**

**Method 2 — Run Dialog:**
1. Press **Windows + R**
2. Type `dnsmgmt.msc`
3. Press **Enter**

**Method 3 — Start Menu:**
1. Search **DNS**
2. Click **DNS Manager**

---

## 1. Installing the DNS Server Role

### GUI — Step by Step
1. Open **Server Manager**
2. Click **Manage** → **Add Roles and Features**
3. Click **Next** through the wizard until **Server Roles**
4. Check **DNS Server**
5. Click **Add Features** when prompted
6. Click **Next** through remaining pages → **Install**
7. Wait for installation to complete → **Close**

### PowerShell
```powershell
# Install DNS Server role
Install-WindowsFeature -Name DNS -IncludeManagementTools

# Verify installation
Get-WindowsFeature -Name DNS
```

---

## 2. Creating a Forward Lookup Zone

A Forward Lookup Zone resolves names to IP addresses for your domain.

### GUI — Step by Step
1. Open **DNS Manager**
2. Expand your server name in the left panel
3. Right-click **Forward Lookup Zones** → **New Zone**
4. The **New Zone Wizard** opens — click **Next**
5. Select zone type:
   - **Primary zone** — the main writable copy (select this for new zones)
   - **Secondary zone** — a read-only copy of another zone
   - **Stub zone** — contains only NS and SOA records
6. Click **Next**
7. If AD-integrated — leave **Store the zone in Active Directory** checked
8. Choose replication scope → **Next**
9. Enter the **Zone name** e.g. `company.com` → **Next**
10. Choose dynamic update setting → **Next** → **Finish**

### PowerShell
```powershell
# Create a primary forward lookup zone
Add-DnsServerPrimaryZone -Name "company.com" -ReplicationScope "Forest" -PassThru

# Create a zone not integrated with AD
Add-DnsServerPrimaryZone -Name "company.com" -ZoneFile "company.com.dns"
```

---

## 3. Creating a Reverse Lookup Zone

A Reverse Lookup Zone resolves IP addresses back to hostnames.

### GUI — Step by Step
1. Open **DNS Manager**
2. Right-click **Reverse Lookup Zones** → **New Zone**
3. Click **Next** through the wizard
4. Select **Primary zone** → **Next**
5. Choose **IPv4 Reverse Lookup Zone** → **Next**
6. Enter the **Network ID** e.g. for 192.168.1.x enter `192.168.1` → **Next**
7. Choose dynamic update setting → **Next** → **Finish**

### PowerShell
```powershell
# Create a reverse lookup zone for 192.168.1.x
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" -ReplicationScope "Forest"
```

---

## 4. Adding DNS Records

### GUI — Adding an A Record (Name to IP)
1. Open **DNS Manager**
2. Expand **Forward Lookup Zones** → click your zone
3. Right-click in the right panel → **New Host (A or AAAA)**
4. Enter:
   - **Name** — hostname e.g. `fileserver`
   - **IP address** — e.g. `192.168.1.100`
   - Check **Create associated pointer (PTR) record** if you have a reverse zone
5. Click **Add Host** → **Done**

### GUI — Adding a CNAME Record (Alias)
1. Right-click in the zone → **New Alias (CNAME)**
2. Enter:
   - **Alias name** — e.g. `files`
   - **Fully qualified domain name (FQDN)** — e.g. `fileserver.company.com`
3. Click **OK**

### GUI — Adding an MX Record (Mail)
1. Right-click in the zone → **New Mail Exchanger (MX)**
2. Enter:
   - **Host or child domain** — leave blank for the root domain
   - **Mail server FQDN** — e.g. `mail.company.com`
   - **Mail server priority** — lower number = higher priority e.g. `10`
3. Click **OK**

### PowerShell
```powershell
# Add A record
Add-DnsServerResourceRecordA -Name "fileserver" -ZoneName "company.com" `
  -IPv4Address "192.168.1.100"

# Add CNAME record
Add-DnsServerResourceRecordCName -Name "files" -ZoneName "company.com" `
  -HostNameAlias "fileserver.company.com"

# Add MX record
Add-DnsServerResourceRecordMX -Name "." -ZoneName "company.com" `
  -MailExchange "mail.company.com" -Preference 10

# Add PTR record (reverse lookup)
Add-DnsServerResourceRecordPtr -Name "100" -ZoneName "1.168.192.in-addr.arpa" `
  -PtrDomainName "fileserver.company.com"
```

---

## 5. Modifying and Deleting DNS Records

### GUI — Step by Step
1. Open **DNS Manager**
2. Expand the zone containing the record
3. Find the record in the right panel
4. To edit — double-click the record → modify → **OK**
5. To delete — right-click the record → **Delete** → **Yes**

### PowerShell
```powershell
# Remove an A record
Remove-DnsServerResourceRecord -ZoneName "company.com" -RRType "A" -Name "fileserver"

# View all records in a zone
Get-DnsServerResourceRecord -ZoneName "company.com"

# View specific record type
Get-DnsServerResourceRecord -ZoneName "company.com" -RRType "A"
```

---

## 6. Configuring Forwarders

Forwarders are DNS servers your server queries when it can't resolve a name locally. Typically these are your ISP's DNS servers or public DNS like Google (8.8.8.8) or Cloudflare (1.1.1.1).

### GUI — Step by Step
1. Open **DNS Manager**
2. Right-click your server name → **Properties**
3. Click the **Forwarders** tab
4. Click **Edit**
5. Enter forwarder IP addresses one at a time:
   - `8.8.8.8` (Google)
   - `8.8.4.4` (Google secondary)
   - `1.1.1.1` (Cloudflare)
6. Click **OK** → **Apply** → **OK**

### PowerShell
```powershell
# Add forwarders
Add-DnsServerForwarder -IPAddress "8.8.8.8","8.8.4.4","1.1.1.1"

# View current forwarders
Get-DnsServerForwarder

# Remove a forwarder
Remove-DnsServerForwarder -IPAddress "8.8.8.8"
```

---

## 7. Testing DNS Resolution

### GUI — Step by Step
1. In **DNS Manager** click **Action** → **Launch nslookup**
2. Type a hostname to resolve e.g. `google.com`
3. Type an IP to reverse resolve e.g. `8.8.8.8`
4. Type `exit` to close

### Command Line / PowerShell
```powershell
# Basic DNS lookup
nslookup google.com

# Lookup against a specific DNS server
nslookup google.com 8.8.8.8

# Reverse lookup
nslookup 8.8.8.8

# PowerShell DNS resolution
Resolve-DnsName -Name "google.com"
Resolve-DnsName -Name "google.com" -Server "8.8.8.8"
Resolve-DnsName -Name "google.com" -Type MX

# Flush DNS cache on a client
ipconfig /flushdns

# View DNS cache
ipconfig /displaydns

# View DNS client settings
Get-DnsClientServerAddress
```

---

## 8. Viewing and Clearing the DNS Server Cache

### GUI — Step by Step
1. Open **DNS Manager**
2. Click **View** → **Advanced** to show the cache
3. Expand your server → **Cached Lookups**
4. Browse cached records
5. To clear — right-click **Cached Lookups** → **Clear Cache**

### PowerShell
```powershell
# Clear DNS server cache
Clear-DnsServerCache -Force

# View server cache
Get-DnsServerCache
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Can't resolve external names | Forwarders not configured or incorrect | Add public DNS forwarders (8.8.8.8, 1.1.1.1) |
| Can't resolve internal names | Record missing or wrong zone | Check DNS records exist and zone is correct |
| DNS changes not taking effect | TTL caching | Wait for TTL to expire or flush DNS cache with `ipconfig /flushdns` |
| Clients using wrong DNS server | DHCP giving wrong DNS address | Check DHCP scope options — DNS server address |
| Slow DNS resolution | Forwarders unreachable | Test forwarder connectivity, try different forwarder |
| Duplicate records | Manual and dynamic registration conflict | Clean up duplicate records, check dynamic update settings |

---

## Quick Reference

```powershell
# View all zones
Get-DnsServerZone

# View all records in a zone
Get-DnsServerResourceRecord -ZoneName "company.com"

# Add A record
Add-DnsServerResourceRecordA -Name "hostname" -ZoneName "company.com" -IPv4Address "192.168.1.x"

# Test resolution
Resolve-DnsName -Name "hostname.company.com"
nslookup hostname.company.com

# Flush client DNS cache
ipconfig /flushdns

# Clear server cache
Clear-DnsServerCache -Force

# View forwarders
Get-DnsServerForwarder
```

---

## Notes
- DNS is tightly integrated with Active Directory — the AD domain name must have a DNS zone
- Always use static IPs on DNS servers — a changing IP breaks name resolution for the whole network
- Set clients to use your internal DNS server first, then a secondary — never only an external DNS
- TTL values control how long records are cached — lower TTL means faster propagation of changes but more DNS traffic
- Dynamic DNS updates allow computers to automatically register their own A records

---

## Related Documents
- [DHCP Configuration](dhcp-configuration.md)
- [Active Directory Domain Configuration](active-directory-domain-configuration.md)
- [Network Security Basics](../Security/network-security-basics.md)
