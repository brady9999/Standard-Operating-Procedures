# WiFi Security Basics
> A guide to wireless security protocols, their vulnerabilities, and which to use in each environment.

**Category:** Security  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **WEP** | Wired Equivalent Privacy | The original wireless security protocol — completely broken |
| **WPA** | WiFi Protected Access | Replaced WEP — also broken, do not use |
| **WPA2** | WiFi Protected Access 2 | Current standard — still widely used and acceptable |
| **WPA3** | WiFi Protected Access 3 | Newest standard — strongest available |
| **PSK** | Pre-Shared Key | A shared password used by all users on the network |
| **SAE** | Simultaneous Authentication of Equals | WPA3's replacement for PSK — resistant to offline cracking |
| **802.1X** | IEEE 802.1X | Port-based access control using individual credentials per user |
| **RADIUS** | Remote Authentication Dial-In User Service | The server that handles 802.1X authentication |
| **EAP** | Extensible Authentication Protocol | A framework for 802.1X authentication methods |
| **PEAP** | Protected EAP | A common EAP method using a server certificate |
| **EAP-TLS** | EAP Transport Layer Security | The strongest EAP method — uses certificates on both client and server |
| **SSID** | Service Set Identifier | The name of the wireless network |
| **BSSID** | Basic Service Set Identifier | The MAC address of the access point radio |
| **PMF** | Protected Management Frames | A WPA3 feature protecting management frames from spoofing |
| **OWE** | Opportunistic Wireless Encryption | WPA3 feature providing encryption on open networks |
| **PMKID** | Pairwise Master Key Identifier | A value in the handshake that can be used for offline cracking attacks |
| **TKIP** | Temporal Key Integrity Protocol | WPA's encryption — broken, avoid |
| **CCMP** | Counter Mode CBC-MAC Protocol | WPA2's encryption using AES — secure |
| **Evil Twin** | Evil Twin Attack | A rogue AP mimicking a legitimate network to intercept traffic |
| **Deauth** | Deauthentication Attack | Forcing clients off a network by sending spoofed disconnect frames |
| **Captive Portal** | Captive Portal | A web page requiring acceptance or login before granting network access |

---

## Overview
Wireless security has evolved significantly since WiFi was first introduced. Choosing the wrong protocol or configuration leaves your network open to attacks that can be executed from the parking lot. The goal is to select the strongest protocol your environment supports and segment wireless traffic appropriately.

**Security ranking from weakest to strongest:**
```
Open (no security)   ← Never use in enterprise
WEP                  ← Completely broken — never use
WPA-TKIP             ← Broken — never use
WPA2-TKIP            ← Avoid — TKIP is weak
WPA2-CCMP (AES)      ← Acceptable — current minimum standard
WPA3-SAE             ← Recommended for new deployments
WPA3-Enterprise      ← Strongest — use in high-security environments
```

---

## 1. Protocol Deep Dive

### WEP — Wired Equivalent Privacy (1997)
**Status: Completely broken — never use**

WEP was the original wireless security standard. It uses RC4 encryption with a static key shared by all users. It was cracked in 2001 and can be broken in minutes with freely available tools.

**Vulnerabilities:**
- Static encryption keys — never change
- Weak IV (Initialization Vector) — only 24 bits, collisions are trivial
- Can be cracked in under 5 minutes with tools like Aircrack-ng
- No protection against packet forgery or replay attacks

**When to use:** Never. Under no circumstances.

---

### WPA — WiFi Protected Access (2003)
**Status: Broken — never use**

WPA was a stopgap replacement for WEP while WPA2 was being finalized. It used TKIP encryption which has known vulnerabilities.

**Vulnerabilities:**
- TKIP encryption is broken — KRACK and other attacks
- Still uses RC4 under the hood
- Can be cracked faster than WPA2

**When to use:** Never. Only exists for compatibility with very old hardware.

---

### WPA2 — WiFi Protected Access 2 (2004)
**Status: Acceptable — current minimum standard**

WPA2 replaced WPA and introduced AES-CCMP encryption which is still considered secure. It comes in two modes:

**WPA2-Personal (PSK):**
- Uses a single shared password for all users
- Password can be cracked offline if an attacker captures the 4-way handshake
- Vulnerable to PMKID attack — no handshake even needed
- Strength depends entirely on password complexity
- **Use for:** Home networks, small offices, guest networks with short-lived passwords

**WPA2-Enterprise (802.1X):**
- Each user authenticates with their own credentials
- No shared password — compromise of one user doesn't affect others
- Requires a RADIUS server
- **Use for:** Corporate environments, university networks, anywhere user accountability matters

**WPA2 Vulnerabilities:**
- PMKID attack — allows offline cracking without capturing a handshake
- KRACK (Key Reinstallation Attack) — patched in 2017, ensure all devices are updated
- Dictionary/brute force attacks against weak PSK passwords
- No protection for management frames (deauth attacks work)

---

### WPA3 — WiFi Protected Access 3 (2018)
**Status: Recommended — use whenever possible**

WPA3 addresses WPA2's major weaknesses. It comes in two modes:

**WPA3-Personal (SAE):**
- SAE replaces PSK — uses a Dragonfly handshake
- Resistant to offline dictionary attacks — even if you capture the handshake you can't crack it offline
- Forward secrecy — past sessions can't be decrypted even if the password is later compromised
- **Use for:** Home networks, small offices, anywhere WPA2-Personal was used

**WPA3-Enterprise:**
- Requires 192-bit encryption (CNSA Suite)
- Mandatory PMF (Protected Management Frames)
- **Use for:** Government, defense, healthcare, high-security environments

**WPA3 Transition Mode:**
- Allows WPA2 and WPA3 devices to coexist on the same network
- Use this during migration periods

**WPA3 Vulnerabilities:**
- Dragonblood attacks (2019) — partially patched, update your APs
- Side-channel attacks against SAE — mitigated in updated implementations
- Still relatively new — implementation bugs possible in older firmware

---

### Open Networks with OWE (Enhanced Open)
**Status: Use for guest/public networks instead of truly open**

Traditional open networks send all traffic unencrypted. OWE (Opportunistic Wireless Encryption) is a WPA3 feature that provides encryption on open networks without requiring a password.

- Each client gets a unique encryption key
- Protects against passive eavesdropping
- Does NOT authenticate the network — still vulnerable to evil twin attacks
- **Use for:** Guest networks, coffee shops, conferences — anywhere you need open access but want basic encryption

---

## 2. Personal vs Enterprise Mode

### Personal Mode (PSK/SAE)

```
Client ──────────── Access Point
         Password
         (shared by everyone)
```

**Pros:**
- Simple to set up
- No additional infrastructure needed
- Works for small environments

**Cons:**
- Single compromised password = entire network exposed
- No per-user accountability — can't tell which device did what
- Rotating the password requires updating every device
- Vulnerable to offline cracking (WPA2) or inside threats

**Best practices for PSK networks:**
- Use WPA3-SAE or WPA2/WPA3 transition mode
- Use a strong random password (20+ characters, all character types)
- Rotate the password periodically (quarterly for corporate, monthly for high-risk)
- Use separate SSIDs for different user groups
- Never share the corporate WiFi password with guests

---

### Enterprise Mode (802.1X/RADIUS)

```
Client ──────────── Access Point ──────────── RADIUS Server ──────────── Active Directory
         Username/Password                    Validates credentials        User database
         or Certificate
```

**Pros:**
- Each user has unique credentials
- Compromise of one user doesn't affect others
- Accountability — logs show exactly which user connected when
- Integration with Active Directory — use domain credentials
- Certificates can replace passwords entirely (EAP-TLS)
- Automatic revocation — disable AD account = no WiFi access

**Cons:**
- Requires RADIUS server infrastructure
- More complex to configure
- Certificate management adds overhead (EAP-TLS)

**EAP Methods — Strongest to Weakest:**

| Method | Auth Method | Certificate on Client | Security Level |
|--------|-------------|----------------------|----------------|
| **EAP-TLS** | Mutual certificates | Yes — required | Highest |
| **PEAP-MSCHAPv2** | Server cert + username/password | No | High |
| **EAP-TTLS** | Server cert + username/password | No | High |
| **LEAP** | Username/password only | No | Broken — avoid |
| **EAP-MD5** | Password only | No | Broken — avoid |

---

## 3. Which Protocol for Each Environment

### Home Network
```
Recommended: WPA3-Personal (SAE)
Fallback:    WPA2-Personal (AES/CCMP) if WPA3 not supported

Password: 20+ random characters
SSID: Don't use your name or address
Guest network: Separate SSID, WPA3 or WPA2, isolated from main network
IoT devices: Third SSID, isolated VLAN
```

### Small Office (< 50 users)
```
Recommended: WPA3-Personal or WPA2/WPA3 transition mode
             Separate SSIDs for staff and guests

Staff SSID:   WPA3/WPA2, strong rotating password, isolated from guest
Guest SSID:   WPA3/WPA2, separate VLAN, internet only — no LAN access
IoT SSID:     WPA3/WPA2, separate isolated VLAN

If budget allows: WPA2/WPA3-Enterprise with Windows NPS as RADIUS
```

### Corporate Environment (50+ users)
```
Recommended: WPA2-Enterprise or WPA3-Enterprise
             802.1X with PEAP-MSCHAPv2 (minimum)
             EAP-TLS with certificates for highest security

Staff SSID:   WPA3-Enterprise, 802.1X, VLAN per department
Guest SSID:   WPA3-Personal or OWE, isolated VLAN, captive portal
BYOD SSID:    WPA3-Enterprise, 802.1X, separate VLAN, limited access
IoT SSID:     WPA2-Personal, isolated VLAN, no internet or LAN access
Management:   Wired only if possible, or dedicated secured SSID
```

### Healthcare / Medical
```
Required: WPA3-Enterprise or WPA2-Enterprise
          EAP-TLS preferred (certificates)
          PMF (Protected Management Frames) mandatory
          Strict VLAN segmentation — medical devices on isolated network
          Separate SSID for patient devices with captive portal

Note: HIPAA requires encryption of ePHI in transit — WPA3-Enterprise satisfies this
```

### Government / Defense
```
Required: WPA3-Enterprise with 192-bit mode
          EAP-TLS with smart card or hardware token certificates
          PMF mandatory
          No open or guest networks on classified segments
          Wireless may be prohibited in certain areas
```

### Retail / Point of Sale
```
Required: PCI DSS compliance — separate wireless networks for POS
          POS devices: WPA2/WPA3-Enterprise, isolated VLAN
          Staff: WPA2/WPA3-Enterprise or strong PSK, separate VLAN
          Guest/Customer: OWE or WPA3-Personal, internet only, captive portal
          Never connect POS devices to guest networks
```

---

## 4. Common Wireless Attacks

### Evil Twin Attack
**What it is:** An attacker creates a rogue AP with the same SSID as a legitimate network. Clients connect to the evil twin instead and the attacker intercepts all traffic.

**Prevention:**
- Use 802.1X/Enterprise — clients verify the RADIUS server certificate
- Enable 802.11w (PMF) — protects management frames
- Use EAP-TLS — mutual certificate authentication prevents evil twin
- Train users to verify certificate warnings before connecting

---

### Deauthentication Attack
**What it is:** An attacker sends spoofed 802.11 deauthentication frames forcing clients to disconnect. This can be used to:
- Disrupt service
- Force clients to reconnect (captures handshake for cracking)

**Prevention:**
- Enable **PMF (Protected Management Frames)** — 802.11w
- WPA3 requires PMF — another reason to upgrade
- WPA2 with PMF optional — enable it on all APs

---

### PMKID Attack (WPA2)
**What it is:** An attacker can request the PMKID from an AP without a client being connected. The PMKID can then be used for offline password cracking without ever capturing a 4-way handshake.

**Prevention:**
- Use **WPA3-SAE** — immune to PMKID attacks
- Use a very strong PSK (20+ random characters) if stuck on WPA2
- Use WPA2-Enterprise — no PSK to crack

---

### Password Cracking (WPA2-PSK)
**What it is:** Capturing the 4-way handshake when a client connects, then running dictionary or brute force attacks offline.

**Prevention:**
- Long, complex, random passwords — 20+ characters
- Upgrade to WPA3-SAE — resistant to offline cracking
- Use Enterprise mode — no shared password to crack

---

### Rogue AP
**What it is:** An unauthorized AP connected to the internal network, providing an unauthorized wireless entry point.

**Prevention:**
- Wireless Intrusion Detection/Prevention System (WIDS/WIPS)
- Regular wireless site surveys
- 802.1X on wired switch ports — unauthorized APs can't get on the network
- Monitor for unauthorized SSIDs in your environment

---

## 5. Hardening Recommendations

### Access Point Configuration
```
✅ Use WPA3 or WPA2/WPA3 transition mode
✅ Enable PMF (Protected Management Frames)
✅ Disable WEP and WPA support completely
✅ Disable TKIP — AES/CCMP only
✅ Keep AP firmware updated
✅ Change default AP admin credentials
✅ Restrict AP management to wired management VLAN
✅ Disable remote management from wireless clients
✅ Enable rogue AP detection if your controller supports it
✅ Configure AP in correct channel (non-overlapping: 1, 6, 11 for 2.4GHz)
```

### SSID Design
```
✅ Separate SSIDs for each trust level (staff, guest, IoT)
✅ Each SSID on its own VLAN
✅ Guest VLAN has internet access only — no LAN access
✅ IoT VLAN is fully isolated — no internet, no LAN if possible
✅ Use meaningful SSID names but don't reveal technology (avoid "CompanyWifi-Cisco")

⚠️ Hidden SSIDs — security through obscurity:
  - Hidden SSIDs do NOT improve security
  - Devices still broadcast probe requests looking for the SSID
  - Adds complexity without benefit
  - Only use for specialized non-user-facing networks
  
⚠️ MAC filtering:
  - MAC addresses are easily spoofed
  - Adds administrative overhead without meaningful security
  - Not a substitute for proper authentication
  - Only use as an additional layer, never primary control
```

### Password Policy for PSK Networks
```
Minimum:  16 characters
Recommended: 20+ characters
Character set: Uppercase + lowercase + numbers + symbols
Example generator: openssl rand -base64 24

Rotation schedule:
- High security: Monthly
- Standard corporate: Quarterly
- Low risk: Annually or when compromise suspected

Never use:
- Company name or address
- Common words or phrases
- Sequential numbers
- Previous passwords
```

### Guest Network Best Practices
```
✅ Completely separate VLAN — guests cannot reach internal LAN
✅ Client isolation — guests cannot see each other
✅ Bandwidth limiting — prevent single user consuming all bandwidth
✅ Captive portal — require acceptance of terms
✅ Short DHCP leases — 1-4 hours for guest networks
✅ Logging — capture MAC and IP for compliance
✅ Separate internet uplink if possible (not sharing corporate bandwidth)
✅ Regular password rotation (weekly or monthly)
✅ Time-limited access for visitors
```

---

## 6. Setting Up RADIUS for Enterprise WiFi (Windows NPS)

Windows Network Policy Server (NPS) is a built-in RADIUS server available on Windows Server.

### GUI — Install NPS
1. Open **Server Manager** → **Add Roles and Features**
2. Select **Network Policy and Access Services**
3. Check **Network Policy Server**
4. Install → **Close**

### Configure NPS for WiFi
1. Open **Network Policy Server** from Server Manager → Tools
2. Right-click **RADIUS Clients** → **New**
   - Friendly name: `AP-Office-01`
   - Address: IP of the access point
   - Shared secret: strong random string (same configured on AP)
3. Go to **Policies** → **Network Policies** → **New**
   - Policy name: `Corporate WiFi`
   - Type: **Wireless**
   - Condition: Add **Windows Groups** → add your AD group
   - Authentication: Select **PEAP** → configure server certificate
4. Configure AP to use WPA2/WPA3-Enterprise and point to NPS server IP

### PowerShell — Add RADIUS Client
```powershell
# Add AP as RADIUS client in NPS
New-NpsRadiusClient `
  -Name "AP-Office-01" `
  -Address "192.168.1.20" `
  -SharedSecret "StrongRandomSecret123!"

# View RADIUS clients
Get-NpsRadiusClient

# View NPS logs
Get-WinEvent -LogName "Security" | Where-Object {$_.Id -in 6272,6273,6274} |
  Select-Object TimeCreated, Message -First 20
# 6272 = Network Policy Server granted access
# 6273 = Network Policy Server denied access
```

---

## Quick Reference

### Protocol Selection Chart
```
Environment          Recommended Protocol
─────────────────────────────────────────
Home                 WPA3-Personal
Small Office         WPA3-Personal or WPA2/WPA3-Enterprise
Corporate            WPA3-Enterprise (802.1X + PEAP or EAP-TLS)
Guest Network        WPA3-Personal + client isolation + captive portal
IoT Devices          WPA2-Personal + isolated VLAN
Healthcare           WPA3-Enterprise + EAP-TLS
Government/Defense   WPA3-Enterprise 192-bit + EAP-TLS
Public/Open          WPA3 Enhanced Open (OWE)
```

### What to Never Use
```
❌ WEP — completely broken
❌ WPA (original) — broken
❌ WPA2-TKIP — TKIP is broken
❌ Open networks without OWE
❌ Weak PSK passwords (under 16 characters)
❌ LEAP or EAP-MD5 — both broken
❌ Sharing corporate PSK with guests
❌ Relying on hidden SSIDs for security
❌ MAC filtering as primary security control
```

---

## Notes
- WPA3 is the current standard — push to upgrade APs and clients as hardware refreshes happen
- WPA2-Enterprise with PEAP is significantly more secure than WPA2-Personal regardless of password strength
- PMF (Protected Management Frames) should be enabled on all WPA2 deployments — it's mandatory in WPA3
- The weakest link in wireless security is usually the PSK password or lack of network segmentation
- Wireless security audits using tools like Kismet or Wireshark should be conducted periodically
- For Niche Technology specifically — handling police data means WPA3-Enterprise or WPA2-Enterprise is the expected standard

---

## Related Documents
- [Network Security Basics](network-security-basics.md)
- [ISO 27001 Basics](iso-27001-basics.md)
- [Cisco Access Point Configuration](../Networking/cisco-access-point-configuration.md)
- [Cisco Access Point Troubleshooting](../Hardware/cisco-access-troubleshooting.md)
- [VLAN Configuration](../Networking/vlan-configuration-cisco-switch.md)
