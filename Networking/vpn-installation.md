# VPN Installation & Configuration
> A guide to installing and configuring VPN solutions including Windows Server RRAS, WireGuard, and Cisco site-to-site VPN.

**Category:** Networking  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **VPN** | Virtual Private Network | An encrypted tunnel connecting two networks or a device to a network over the internet |
| **Tunnel** | VPN Tunnel | The encrypted connection between two VPN endpoints |
| **Endpoint** | VPN Endpoint | A device or server that terminates the VPN connection |
| **Client VPN** | Remote Access VPN | A user connecting their device to a corporate network remotely |
| **Site-to-Site VPN** | Site-to-Site VPN | Connecting two entire networks together permanently |
| **Split Tunneling** | Split Tunneling | Only VPN traffic goes through the tunnel — local traffic goes direct |
| **Full Tunneling** | Full Tunnel | All traffic routes through the VPN including internet traffic |
| **RRAS** | Routing and Remote Access Service | Windows Server's built-in VPN server |
| **IPsec** | Internet Protocol Security | A suite of protocols for encrypting and authenticating IP traffic |
| **IKE** | Internet Key Exchange | The protocol used to establish and manage IPsec sessions |
| **L2TP** | Layer 2 Tunneling Protocol | A VPN protocol often used with IPsec for encryption |
| **SSTP** | Secure Socket Tunneling Protocol | Microsoft's VPN protocol using HTTPS (port 443) |
| **WireGuard** | WireGuard | A modern, fast, simple VPN protocol |
| **OpenVPN** | OpenVPN | An open-source VPN protocol using SSL/TLS |
| **Tailscale** | Tailscale | A mesh VPN built on WireGuard — easy to deploy |
| **RADIUS** | Remote Authentication Dial-In User Service | A protocol for authenticating VPN users centrally |
| **PSK** | Pre-Shared Key | A shared secret used for VPN authentication |
| **PKI** | Public Key Infrastructure | Certificate-based authentication for VPN |

---

## Overview
VPNs create encrypted tunnels over public networks (internet) allowing:
- Remote employees to securely access company resources
- Branch offices to connect to headquarters
- Secure communication between cloud and on-premises networks

**VPN Types:**

| Type | Use Case | Protocol |
|------|----------|---------|
| Remote Access | Users connecting from home | L2TP/IPsec, SSTP, WireGuard, OpenVPN |
| Site-to-Site | Office to office | IPsec, GRE over IPsec |
| Mesh VPN | Any device to any device | WireGuard, Tailscale |

---

## Prerequisites
- Static public IP on the VPN server (or dynamic DNS)
- Firewall/router must forward VPN ports to the server
- Windows Server (for RRAS) or Linux (for WireGuard)
- Administrator rights

---

## Option 1 — Windows Server RRAS (Remote Access VPN)

### Installing RRAS

#### GUI — Step by Step
1. Open **Server Manager**
2. Click **Manage** → **Add Roles and Features**
3. Click **Next** until **Server Roles**
4. Check **Remote Access**
5. Click **Next** → under **Role Services** check:
   - **DirectAccess and VPN (RAS)**
   - **Routing**
6. Click **Next** → **Install**
7. After installation click **Open the Getting Started Wizard**
8. Select **Deploy VPN only**
9. Right-click your server → **Configure and Enable Routing and Remote Access**
10. Select **Custom configuration** → **Next**
11. Check **VPN access** → **Next** → **Finish**
12. Click **Start service**

#### PowerShell
```powershell
# Install RRAS role
Install-WindowsFeature -Name RemoteAccess, RSAT-RemoteAccess-PowerShell

# Install VPN and routing
Install-RemoteAccess -VpnType VpnS2S
```

### Configuring L2TP/IPsec VPN
```powershell
# Set the pre-shared key for L2TP
Set-VpnServerIPsecConfiguration -TunnelType L2TP `
  -EncryptionType RequireEncryption `
  -IdleDisconnectSeconds 300

# Set L2TP pre-shared key via registry
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\RemoteAccess\Parameters\IKEV2" `
  -Name "CertificateEKUs" -Value 0
```

### Allowing Users to Connect via VPN

#### GUI — Step by Step
1. Open **Active Directory Users and Computers**
2. Find the user → right-click → **Properties**
3. Click the **Dial-in** tab
4. Under **Network Access Permission** select **Allow access**
5. Click **Apply** → **OK**

#### PowerShell
```powershell
# Allow user VPN access
Set-ADUser -Identity "jsmith" -msNPAllowDialin $true

# Grant VPN access to all users in a group
Get-ADGroupMember -Identity "VPN-Users" | ForEach-Object {
  Set-ADUser -Identity $_.SamAccountName -msNPAllowDialin $true
}
```

### Firewall Ports to Open for RRAS VPN
```
L2TP/IPsec: UDP 500, UDP 4500, UDP 1701
SSTP: TCP 443
PPTP: TCP 1723 (avoid — not secure)
IKEv2: UDP 500, UDP 4500
```

---

## Option 2 — WireGuard VPN (Linux Server)

WireGuard is a modern, fast, and simple VPN. It is the protocol behind Tailscale.

### Installing WireGuard on Ubuntu/Debian
```bash
# Update packages
sudo apt update

# Install WireGuard
sudo apt install wireguard -y

# Enable IP forwarding
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Generating Keys
```bash
# Generate server private and public key
wg genkey | tee /etc/wireguard/server_private.key | wg pubkey > /etc/wireguard/server_public.key

# Generate client private and public key
wg genkey | tee /etc/wireguard/client_private.key | wg pubkey > /etc/wireguard/client_public.key

# View keys
cat /etc/wireguard/server_private.key
cat /etc/wireguard/server_public.key
```

### Server Configuration
```bash
# Create server config file
sudo nano /etc/wireguard/wg0.conf
```

```ini
[Interface]
# Server private key
PrivateKey = SERVER_PRIVATE_KEY_HERE
# VPN IP address for the server
Address = 10.0.0.1/24
# Listening port
ListenPort = 51820
# NAT rules — replace eth0 with your internet interface
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Client public key
PublicKey = CLIENT_PUBLIC_KEY_HERE
# IP address to assign to this client
AllowedIPs = 10.0.0.2/32
```

```bash
# Start WireGuard
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0

# Check status
sudo wg show
```

### Client Configuration
```ini
[Interface]
# Client private key
PrivateKey = CLIENT_PRIVATE_KEY_HERE
# Client VPN IP
Address = 10.0.0.2/24
# Use server as DNS
DNS = 192.168.1.10

[Peer]
# Server public key
PublicKey = SERVER_PUBLIC_KEY_HERE
# Server public IP and port
Endpoint = YOUR.PUBLIC.IP:51820
# Route all traffic through VPN (full tunnel)
AllowedIPs = 0.0.0.0/0
# Or split tunnel — only company network
# AllowedIPs = 192.168.1.0/24
# Keep connection alive
PersistentKeepalive = 25
```

---

## Option 3 — Tailscale (Mesh VPN)

Tailscale is the easiest VPN to deploy — built on WireGuard. Each device gets a persistent IP in the 100.x.x.x range and can reach any other Tailscale device directly.

### Installing Tailscale on Linux
```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Authenticate and connect
sudo tailscale up

# Enable as subnet router (expose local network to other Tailscale devices)
sudo tailscale up --advertise-routes=192.168.1.0/24

# Check status
tailscale status
tailscale ip
```

### Installing Tailscale on Windows
```powershell
# Download and install from https://tailscale.com/download
# Or via winget
winget install tailscale.tailscale

# Authenticate via browser when prompted
# Check status
tailscale status
```

### Tailscale Admin Console
- Manage devices at `https://login.tailscale.com/admin/machines`
- Approve subnet routes, revoke access, view connected devices

---

## Option 4 — Cisco Site-to-Site IPsec VPN

Connects two Cisco routers permanently over the internet.

```ios
! ── On Router 1 (Site A — 192.168.1.0/24) ──

! Step 1: Create ISAKMP Policy (IKE Phase 1)
RTR-SITE-A(config)# crypto isakmp policy 10
RTR-SITE-A(config-isakmp)# encryption aes 256
RTR-SITE-A(config-isakmp)# hash sha256
RTR-SITE-A(config-isakmp)# authentication pre-share
RTR-SITE-A(config-isakmp)# group 14
RTR-SITE-A(config-isakmp)# lifetime 86400
RTR-SITE-A(config-isakmp)# exit

! Step 2: Set pre-shared key (must match on both routers)
RTR-SITE-A(config)# crypto isakmp key VPNSecretKey123 address 203.0.113.5

! Step 3: Create IPsec Transform Set (IKE Phase 2)
RTR-SITE-A(config)# crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
RTR-SITE-A(cfg-crypto-trans)# mode tunnel
RTR-SITE-A(cfg-crypto-trans)# exit

! Step 4: Create ACL defining interesting traffic
RTR-SITE-A(config)# access-list 110 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255

! Step 5: Create crypto map
RTR-SITE-A(config)# crypto map SITE-A-MAP 10 ipsec-isakmp
RTR-SITE-A(config-crypto-map)# set peer 203.0.113.5
RTR-SITE-A(config-crypto-map)# set transform-set TSET
RTR-SITE-A(config-crypto-map)# match address 110
RTR-SITE-A(config-crypto-map)# exit

! Step 6: Apply crypto map to WAN interface
RTR-SITE-A(config)# interface GigabitEthernet 0/1
RTR-SITE-A(config-if)# crypto map SITE-A-MAP
RTR-SITE-A(config-if)# exit

RTR-SITE-A# wr
```

### Verifying Site-to-Site VPN
```ios
! Check IKE Phase 1 (ISAKMP SA)
show crypto isakmp sa

! Check IKE Phase 2 (IPsec SA)
show crypto ipsec sa

! Check crypto map
show crypto map

! Debug VPN (use carefully)
debug crypto isakmp
debug crypto ipsec
undebug all
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| VPN clients can't connect | Firewall blocking VPN ports | Open required ports (500, 4500, 1701 for L2TP) |
| L2TP connection fails | Pre-shared key mismatch | Verify PSK matches on client and server |
| Connected but no network access | Routing not configured | Enable IP forwarding, check NAT rules |
| WireGuard tunnel up but no traffic | AllowedIPs misconfigured | Check AllowedIPs on both sides |
| Tailscale devices can't reach each other | Firewall blocking direct connection | Tailscale will use relay (DERP) — check admin console |
| Site-to-site VPN flapping | NAT on one side changing IP | Configure NAT traversal (NAT-T) |
| Slow VPN speeds | MTU mismatch causing fragmentation | Set MTU to 1400 on VPN interface |

---

## Quick Reference

```bash
# WireGuard — check status
sudo wg show

# WireGuard — start/stop
sudo systemctl start wg-quick@wg0
sudo systemctl stop wg-quick@wg0

# Tailscale — status
tailscale status

# Tailscale — connect
sudo tailscale up

# Tailscale — disconnect
sudo tailscale down
```

```ios
! Cisco — check VPN status
show crypto isakmp sa
show crypto ipsec sa

! Cisco — clear VPN session
clear crypto session
clear crypto isakmp
```

---

## Notes
- WireGuard and Tailscale are the recommended solutions for new deployments — simpler and more secure than PPTP or L2TP
- PPTP is deprecated and should never be used — it has known serious security vulnerabilities
- Always use a strong pre-shared key or certificate-based authentication
- For remote access VPNs always require MFA in addition to a password
- Split tunneling reduces load on the VPN server but allows users' local traffic to bypass corporate security
- Firewall ports vary by protocol — check your specific VPN type's requirements

---

## Related Documents
- [Cisco Router Configuration](cisco-router-configuration.md)
- [Network Security Basics](../Security/network-security-basics.md)
- [Remote Desktop Configuration](../Windows/remote-desktop-configuration.md)
