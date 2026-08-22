# Virtual Machine Routing
> A guide to configuring network routing for virtual machines across different hypervisors and environments.

**Category:** Virtual Machines  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Hypervisor** | Hypervisor | Software that creates and manages virtual machines |
| **vSwitch** | Virtual Switch | A software-based network switch connecting VMs |
| **vNIC** | Virtual Network Interface Card | A virtual network adapter assigned to a VM |
| **Bridge** | Network Bridge | Connects a VM's virtual NIC to a physical network |
| **NAT** | Network Address Translation | Translates VM private IPs to the host's IP for internet access |
| **Host-Only** | Host-Only Network | VMs can only communicate with the host — no external access |
| **VLAN** | Virtual LAN | A logical network segment carried on a trunk |
| **Trunk** | Trunk Port | A connection carrying multiple VLANs |
| **Routing Table** | Routing Table | The list of known networks and how to reach them |
| **Default Gateway** | Default Gateway | The router IP that handles traffic outside the local subnet |
| **IP Forwarding** | IP Forwarding | Allowing a Linux host to route packets between interfaces |
| **SDN** | Software-Defined Networking | Network configuration managed through software |
| **OVS** | Open vSwitch | An open-source virtual switch used in enterprise environments |
| **Bonding** | NIC Bonding | Combining multiple physical NICs for redundancy or performance |

---

## Overview
Virtual machine networking can be complex — VMs need to communicate with each other, the host, and the external network. The network mode chosen determines what the VM can reach.

**Common VM Network Modes:**

| Mode | VM ↔ VM | VM ↔ Host | VM ↔ Internet | Use Case |
|------|---------|----------|--------------|---------|
| **Bridged** | ✅ | ✅ | ✅ | VM acts as a full network citizen |
| **NAT** | ✅ (same NAT) | ✅ | ✅ (via host) | Internet access without exposing VM |
| **Host-Only** | ✅ | ✅ | ❌ | Isolated lab environments |
| **Internal** | ✅ | ❌ | ❌ | Fully isolated VM-to-VM only |

---

## 1. Proxmox VE Networking

Proxmox uses Linux bridges for VM networking. VMs connect to bridges which connect to physical interfaces.

### Proxmox Network Architecture
```
Physical NIC (eno1)
└── Linux Bridge (vmbr0)
    ├── VM 100 (vNIC: eth0)
    ├── VM 101 (vNIC: eth0)
    └── VM 102 (vNIC: eth0)
```

### Viewing Network Configuration
```bash
# View all network interfaces on Proxmox host
ip addr
ip link show

# View bridge configuration
brctl show

# View routing table
ip route

# View Proxmox network config file
cat /etc/network/interfaces
```

### Creating a Linux Bridge in Proxmox

**GUI — Step by Step:**
1. Go to **Proxmox Web UI** → select the node
2. Click **System** → **Network**
3. Click **Create** → **Linux Bridge**
4. Set:
   - **Name**: `vmbr0` (or `vmbr1` for additional bridges)
   - **IPv4/CIDR**: IP for host management e.g. `192.168.1.10/24`
   - **Gateway**: `192.168.1.1`
   - **Bridge ports**: `eno1` (the physical NIC)
   - **VLAN aware**: Check this if using VLANs
5. Click **Create**

**CLI — /etc/network/interfaces:**
```bash
# Edit network config
nano /etc/network/interfaces
```

```
# Bridge for main network
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0

# Second bridge for isolated lab network
auto vmbr1
iface vmbr1 inet static
    address 10.0.0.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
```

```bash
# Apply network changes
ifreload -a
# Or restart networking
systemctl restart networking
```

### VLAN-Aware Bridge in Proxmox

```bash
# Enable VLAN awareness on the bridge
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094
```

```bash
# Assign a VM to a specific VLAN via GUI:
# VM → Hardware → Network Device → VLAN Tag: 10

# Or via CLI
qm set 100 -net0 virtio,bridge=vmbr0,tag=10
```

### Configuring a VM as a Router in Proxmox

```bash
# Enable IP forwarding on the VM (Linux)
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Add routing rules inside the VM
ip route add 192.168.2.0/24 via 10.0.0.1

# Make persistent
echo "192.168.2.0/24 via 10.0.0.1" >> /etc/network/routes
```

---

## 2. Hyper-V Networking

Hyper-V uses virtual switches to connect VMs to networks.

### Virtual Switch Types
| Type | VM Access | Host Access | External Access |
|------|----------|------------|----------------|
| External | ✅ | ✅ | ✅ |
| Internal | ✅ | ✅ | ❌ |
| Private | ✅ | ❌ | ❌ |

### PowerShell — Managing Hyper-V Networking
```powershell
# List all virtual switches
Get-VMSwitch

# Create an external switch
New-VMSwitch -Name "ExternalSwitch" -NetAdapterName "Ethernet" -AllowManagementOS $true

# Create an internal switch
New-VMSwitch -Name "InternalSwitch" -SwitchType Internal

# Create a private switch
New-VMSwitch -Name "PrivateSwitch" -SwitchType Private

# View VM network adapters
Get-VMNetworkAdapter -VMName "MyVM"

# Connect VM to a switch
Connect-VMNetworkAdapter -VMName "MyVM" -SwitchName "ExternalSwitch"

# Add a second NIC to a VM
Add-VMNetworkAdapter -VMName "MyVM" -SwitchName "InternalSwitch"

# Remove a NIC from a VM
Remove-VMNetworkAdapter -VMName "MyVM" -Name "Network Adapter 2"
```

### Hyper-V VLAN Configuration
```powershell
# Set VLAN on a VM NIC
Set-VMNetworkAdapterVlan -VMName "MyVM" -Access -VlanId 10

# Set trunk mode (allows multiple VLANs)
Set-VMNetworkAdapterVlan -VMName "MyVM" -Trunk `
  -AllowedVlanIdList "10,20,30" `
  -NativeVlanId 1

# View VLAN config
Get-VMNetworkAdapterVlan -VMName "MyVM"
```

---

## 3. VMware Workstation/ESXi Networking

### VMware Network Types
| Network | Description |
|---------|-------------|
| **Bridged (VMnet0)** | VM gets its own IP on physical network |
| **NAT (VMnet8)** | VM shares host IP, hidden behind NAT |
| **Host-Only (VMnet1)** | VM communicates with host only |
| **Custom VMnet** | Custom isolated networks |

### ESXi vSwitch Configuration (PowerCLI)
```powershell
# Connect to ESXi
Connect-VIServer -Server esxi.company.com

# Create a virtual switch
New-VirtualSwitch -Name "vSwitch1" -NumPorts 24

# Add a port group
New-VirtualPortGroup -Name "VLAN10-Production" -VirtualSwitch "vSwitch1" -VLanId 10

# Assign VM to port group
Get-VM "MyVM" | Get-NetworkAdapter | Set-NetworkAdapter -PortGroup "VLAN10-Production"

# List virtual switches
Get-VirtualSwitch

# List port groups
Get-VirtualPortGroup
```

---

## 4. VM Routing Scenarios

### Scenario 1 — VMs on Different Subnets Communicating

```
VM1: 192.168.10.5/24  (VLAN 10)
VM2: 192.168.20.5/24  (VLAN 20)
Router VM: eth0=192.168.10.1, eth1=192.168.20.1
```

```bash
# On the Router VM — enable IP forwarding
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# On VM1 — add route to reach VM2's subnet
ip route add 192.168.20.0/24 via 192.168.10.1

# On VM2 — add route to reach VM1's subnet
ip route add 192.168.10.0/24 via 192.168.20.1

# Make routes persistent (Ubuntu/Debian)
# Add to /etc/netplan/00-installer-config.yaml or /etc/network/interfaces
```

### Scenario 2 — VM Internet Access via NAT

```bash
# On Linux host — enable NAT for VMs
# Assuming vmbr1 is the internal bridge and ens18 is the internet-connected interface

# Enable IP forwarding
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Add NAT rule
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o ens18 -j MASQUERADE
iptables -A FORWARD -i vmbr1 -o ens18 -j ACCEPT
iptables -A FORWARD -i ens18 -o vmbr1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Make iptables rules persistent
apt install iptables-persistent
netfilter-persistent save
```

### Scenario 3 — Isolated Lab Network

```bash
# Create an isolated internal bridge in Proxmox
auto vmbr2
iface vmbr2 inet static
    address 172.16.0.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

# VMs on vmbr2 can only communicate with each other and the host
# No external access unless NAT is configured
```

---

## 5. VM Network Troubleshooting

```bash
# Check VM network interface
ip addr
ip link show

# Check routing table inside VM
ip route
route -n

# Test connectivity to gateway
ping 192.168.1.1

# Test DNS resolution
ping google.com
nslookup google.com

# Test specific port
nc -zv 192.168.1.1 22
telnet 192.168.1.1 80

# Capture packets on VM interface
tcpdump -i eth0 -n
tcpdump -i eth0 host 192.168.1.1
tcpdump -i eth0 port 80

# View ARP table
arp -n
ip neigh

# Check network statistics
netstat -s
ss -s
```

### On the Proxmox Host
```bash
# Check bridge status
brctl show

# View which VMs are on which bridge
ip link show type bridge_slave

# Check packet flow on bridge
tcpdump -i vmbr0 -n

# View VM network config
qm config 100 | grep net
```

---

## 6. Network Performance Tuning

```bash
# Use VirtIO network drivers for best performance in Linux VMs
# In Proxmox VM config:
qm set 100 -net0 virtio,bridge=vmbr0

# Enable jumbo frames (MTU 9000) for high-throughput environments
ip link set eth0 mtu 9000

# Check current MTU
ip link show eth0 | grep mtu

# Enable TCP offloading (usually on by default)
ethtool -K eth0 tx on rx on gso on tso on

# Check offload settings
ethtool -k eth0
```

---

## Quick Reference

```bash
# View interfaces
ip addr
ip link show

# View routing table
ip route

# Add a static route
ip route add 192.168.2.0/24 via 192.168.1.1

# Add permanent route (Ubuntu netplan)
# Edit /etc/netplan/config.yaml and add routes section

# Enable IP forwarding
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# NAT for VMs
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# Proxmox — view VM network config
qm config [VMID] | grep net

# Hyper-V — list switches
Get-VMSwitch

# Hyper-V — connect VM to switch
Connect-VMNetworkAdapter -VMName "VM" -SwitchName "Switch"
```

---

## Notes
- Always use **VirtIO** drivers for Linux VMs on Proxmox or KVM — they offer significantly better performance than emulated network cards
- **Bridged networking** is simplest for VMs that need to appear as separate network devices
- **NAT** is best for VMs that need internet access but shouldn't be directly reachable from the network
- VLAN tagging in VMs requires the physical switch port connected to the host to be a trunk port
- IP forwarding must be enabled on any VM acting as a router — it's disabled by default on Linux
- Always test VM network connectivity after making changes — use ping and traceroute to verify routing

---

## Related Documents
- [Virtual Networks Basics](virtual-networks-basics.md)
- [Virtual Machine Security Basics](virtual-machine-security-basics.md)
- [Hyper-V Installation](../Windows/hyperv-installation.md)
- [VLAN Configuration](../Networking/vlan-configuration-cisco-switch.md)
