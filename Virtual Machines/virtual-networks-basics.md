# Virtual Networks Basics
> A guide to understanding and configuring virtual networks across Proxmox, Hyper-V, VMware, and cloud environments.

**Category:** Virtual Machines  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **vSwitch** | Virtual Switch | A software-based network switch connecting VMs |
| **vNIC** | Virtual NIC | A virtual network adapter inside a VM |
| **Bridge** | Linux Network Bridge | A Layer 2 device connecting VM traffic to physical networks |
| **Overlay Network** | Overlay Network | A virtual network built on top of a physical network using encapsulation |
| **Underlay** | Underlay Network | The physical network that carries overlay traffic |
| **VXLAN** | Virtual Extensible LAN | An overlay protocol that extends VLANs across routed networks |
| **SDN** | Software-Defined Networking | Network configuration managed through software/APIs |
| **Trunk** | 802.1Q Trunk | A link carrying multiple VLANs |
| **Bonding** | NIC Bonding / Teaming | Combining multiple NICs for redundancy or bandwidth |
| **LACP** | Link Aggregation Control Protocol | Protocol for bonding multiple links |
| **OVS** | Open vSwitch | An open-source virtual switch with advanced features |
| **DPDK** | Data Plane Development Kit | High-performance networking library for SR-IOV and fast packet processing |
| **SR-IOV** | Single Root I/O Virtualization | Hardware feature allowing a physical NIC to appear as multiple virtual NICs |
| **MTU** | Maximum Transmission Unit | Maximum packet size — standard is 1500, jumbo frames are 9000 |
| **Promiscuous Mode** | Promiscuous Mode | A NIC mode that receives all traffic on the network segment |

---

## Overview
Virtual networks allow VMs to communicate with each other, the host, and the physical network. Understanding virtual network types and their behavior is essential for proper VM deployment and troubleshooting.

**Virtual Network Stack:**
```
Physical World:
  Physical NIC (ens18) ← Physical switch ← Other physical devices

Host Layer:
  Linux Bridge (vmbr0) ← Connects VMs to physical NIC
  or
  Open vSwitch (ovs-br0) ← More advanced virtual switching

VM Layer:
  VM vNIC (eth0) ← Connected to the bridge/vSwitch
```

---

## 1. Linux Bridge Networking (Proxmox)

The Linux bridge is the default networking method in Proxmox — simple and reliable.

### How It Works
```
Physical NIC (ens18)
         │
    Linux Bridge (vmbr0)  ← 192.168.1.10/24 (host management IP)
    ┌────┴────┐
   VM100     VM101        ← VMs get IPs in the same or different subnet
  (eth0)    (eth0)
```

### Configuration

```bash
# /etc/network/interfaces

# Physical interface — no IP (bridge member only)
auto ens18
iface ens18 inet manual

# Bridge connected to physical network
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    dns-nameservers 192.168.1.10
    bridge-ports ens18
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes    # Enable if using VLANs

# Internal bridge — no physical NIC (VM-to-VM only)
auto vmbr1
iface vmbr1 inet static
    address 10.0.0.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
```

```bash
# View bridge status
brctl show

# View bridge members
bridge link show

# View MAC table of bridge
bridge fdb show vmbr0

# Apply config changes
ifreload -a
```

### Multiple Bridges for Network Segmentation

```bash
# Production bridge (connected to physical network)
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    bridge-ports ens18
    bridge-stp off
    bridge-fd 0

# Isolated lab bridge (no physical connection)
auto vmbr1
iface vmbr1 inet static
    address 10.10.0.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0

# DMZ bridge (connected to second NIC)
auto vmbr2
iface vmbr2 inet manual
    bridge-ports ens19
    bridge-stp off
    bridge-fd 0
```

---

## 2. VLAN Configuration in Virtual Networks

VLANs allow multiple logical networks to share the same physical infrastructure.

### VLAN-Aware Bridge (Proxmox)

```bash
# Enable VLAN awareness on bridge
auto vmbr0
iface vmbr0 inet static
    address 192.168.99.10/24    # Management IP on native VLAN
    gateway 192.168.99.1
    bridge-ports ens18
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 2-4094          # Allow all VLANs
```

```bash
# Assign VLAN to a VM
qm set 100 -net0 virtio,bridge=vmbr0,tag=10    # VM on VLAN 10
qm set 101 -net0 virtio,bridge=vmbr0,tag=20    # VM on VLAN 20
qm set 102 -net0 virtio,bridge=vmbr0,tag=30    # VM on VLAN 30

# VM with multiple VLANs (trunk to VM)
qm set 103 -net0 virtio,bridge=vmbr0,trunks=10;20;30

# View VM network configuration
qm config 100 | grep net
```

### Physical Switch Port Configuration

The switch port connecting the Proxmox host must be configured as a trunk for VLAN-aware setups.

```ios
! On the Cisco switch
SW-CORE-01(config)# interface GigabitEthernet 0/1
SW-CORE-01(config-if)# description Proxmox-Host-01
SW-CORE-01(config-if)# switchport mode trunk
SW-CORE-01(config-if)# switchport trunk allowed vlan 10,20,30,99
SW-CORE-01(config-if)# switchport trunk native vlan 99
SW-CORE-01(config-if)# no shutdown
```

---

## 3. NIC Bonding (Link Aggregation)

Bonding combines multiple physical NICs for redundancy (failover) or increased bandwidth.

### Bonding Modes

| Mode | Name | Description |
|------|------|-------------|
| 0 | balance-rr | Round-robin — load balances across all NICs |
| 1 | active-backup | One active NIC, others standby — failover only |
| 2 | balance-xor | XOR-based load balancing |
| 4 | 802.3ad (LACP) | IEEE standard link aggregation — requires switch support |
| 5 | balance-tlb | Adaptive transmit load balancing |
| 6 | balance-alb | Adaptive load balancing |

**Most common:** Mode 1 (active-backup) for simple redundancy, Mode 4 (LACP) for bandwidth + redundancy.

### Configuring Bond in Proxmox

```bash
# /etc/network/interfaces

# Bond using active-backup (mode 1)
auto bond0
iface bond0 inet manual
    bond-slaves ens18 ens19
    bond-miimon 100
    bond-mode active-backup

# Or LACP (mode 4) — requires switch to have LACP enabled
auto bond0
iface bond0 inet manual
    bond-slaves ens18 ens19
    bond-miimon 100
    bond-mode 802.3ad
    bond-lacp-rate fast
    bond-xmit-hash-policy layer2+3

# Bridge on top of bond
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    bridge-ports bond0
    bridge-stp off
    bridge-fd 0
```

```bash
# Check bond status
cat /proc/net/bonding/bond0

# View bond interface
ip link show bond0
```

---

## 4. Hyper-V Virtual Networking

### Virtual Switch Types

```powershell
# List all virtual switches
Get-VMSwitch | Select-Object Name, SwitchType, NetAdapterInterfaceDescription

# Create external switch (VMs get physical network access)
New-VMSwitch -Name "External-Production" `
  -NetAdapterName "Ethernet" `
  -AllowManagementOS $true

# Create internal switch (VMs and host communicate)
New-VMSwitch -Name "Internal-Lab" -SwitchType Internal

# Create private switch (VM-to-VM only)
New-VMSwitch -Name "Private-Isolated" -SwitchType Private

# Delete a switch
Remove-VMSwitch -Name "OldSwitch" -Force
```

### Hyper-V VLAN Configuration

```powershell
# Set VM NIC to access mode (single VLAN)
Set-VMNetworkAdapterVlan -VMName "WebServer" -Access -VlanId 10

# Set VM NIC to trunk mode (multiple VLANs)
Set-VMNetworkAdapterVlan -VMName "RouterVM" -Trunk `
  -AllowedVlanIdList "10,20,30,99" `
  -NativeVlanId 99

# View VLAN settings
Get-VMNetworkAdapterVlan -VMName "WebServer"

# Remove VLAN tag (untagged)
Set-VMNetworkAdapterVlan -VMName "WebServer" -Untagged
```

### Hyper-V NIC Teaming

```powershell
# Create NIC team on Hyper-V host
New-NetLbfoTeam -Name "Host-Team" `
  -TeamMembers "Ethernet","Ethernet 2" `
  -TeamingMode SwitchIndependent `
  -LoadBalancingAlgorithm Dynamic

# View team status
Get-NetLbfoTeam

# Create virtual switch on top of team
New-VMSwitch -Name "External-Team" -NetAdapterName "Host-Team"
```

---

## 5. VMware Virtual Networking

### Standard vSwitch (vSS)

```powershell
# Connect to ESXi
Connect-VIServer -Server esxi.company.com

# Create standard vSwitch
New-VirtualSwitch -Host (Get-VMHost) -Name "vSwitch1" -NumPorts 120

# Add uplink to vSwitch
Add-VirtualSwitchPhysicalNetworkAdapter `
  -VirtualSwitch (Get-VirtualSwitch -Name "vSwitch1") `
  -VMHostPhysicalNic (Get-VMHostNetworkAdapter -Physical -Name "vmnic1")

# Create port group
New-VirtualPortGroup -Name "VLAN10-Production" `
  -VirtualSwitch "vSwitch1" `
  -VLanId 10

# Assign VM to port group
Get-VM "MyVM" | Get-NetworkAdapter | Set-NetworkAdapter -PortGroup "VLAN10-Production"

# View vSwitches
Get-VirtualSwitch

# View port groups
Get-VirtualPortGroup
```

---

## 6. Open vSwitch (OVS)

Open vSwitch is an advanced virtual switch used in enterprise and SDN environments.

```bash
# Install OVS
apt install openvswitch-switch -y

# Create a bridge
ovs-vsctl add-br ovs-br0

# Add physical NIC to bridge
ovs-vsctl add-port ovs-br0 ens18

# Add an internal port (for host connectivity)
ovs-vsctl add-port ovs-br0 vlan10 -- set Interface vlan10 type=internal
ip addr add 192.168.10.1/24 dev vlan10
ip link set vlan10 up

# Configure VLAN on a port
ovs-vsctl set port vlan10 tag=10

# Create a trunk port
ovs-vsctl set port ens18 trunks=10,20,30

# View OVS configuration
ovs-vsctl show

# View bridge details
ovs-ofctl show ovs-br0

# View flows (OpenFlow rules)
ovs-ofctl dump-flows ovs-br0

# Add a flow rule
ovs-ofctl add-flow ovs-br0 "priority=100,ip,nw_dst=192.168.10.0/24,actions=output:1"

# Delete OVS bridge
ovs-vsctl del-br ovs-br0
```

---

## 7. Network Performance in Virtual Environments

### VirtIO vs Emulated NICs

```
Emulated NIC (e1000, rtl8139):
- Software emulates legacy hardware
- High CPU overhead
- Slower throughput
- Use only for OS compatibility (Windows XP, legacy)

VirtIO NIC (virtio-net):
- Para-virtualized driver
- Much lower CPU overhead
- Near native throughput
- Requires VirtIO drivers (included in modern Linux, available for Windows)
```

```bash
# Set VM to use VirtIO (Proxmox)
qm set 100 -net0 virtio,bridge=vmbr0

# Or in VM creation:
qm create 100 --net0 virtio,bridge=vmbr0
```

### Jumbo Frames

```bash
# Enable jumbo frames (MTU 9000) for high-throughput environments
# On Proxmox host
ip link set vmbr0 mtu 9000
ip link set ens18 mtu 9000

# Make persistent in /etc/network/interfaces
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    bridge-ports ens18
    mtu 9000

# Inside VM
ip link set eth0 mtu 9000

# Test MTU — send large packets
ping -M do -s 8972 192.168.1.1
# If it fails — MTU mismatch somewhere in the path
```

---

## 8. Troubleshooting Virtual Networks

```bash
# On Proxmox host — check bridge
brctl show vmbr0
bridge link show

# Check if VM is passing traffic
tcpdump -i vmbr0 -n host 192.168.1.50

# Check VM network config (inside the VM)
ip addr
ip route
ip neigh

# Test connectivity inside VM
ping 192.168.1.1          # Gateway
ping 8.8.8.8              # Internet
nslookup google.com       # DNS

# Check VM NIC status in Proxmox
qm config 100 | grep net

# Check if VM's NIC is connected to bridge
ip link show | grep -A2 "tap100i0"

# For VLAN issues — check VLAN tag on VM NIC
qm config 100 | grep tag

# Check physical switch port
# show interfaces trunk
# show vlan brief

# Capture traffic on a specific VLAN
tcpdump -i vmbr0.10 -n

# Test VLAN connectivity
ping -I vlan10 192.168.10.1
```

### Common Virtual Network Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| VM has no network | NIC not connected to bridge | Check `qm config`, reconnect NIC |
| VM can reach host but not network | Bridge not connected to physical NIC | Add physical NIC as bridge port |
| VLAN traffic not passing | VLAN tag mismatch or switch not trunking | Check VM VLAN tag and switch trunk config |
| Slow network performance | Using emulated NIC instead of VirtIO | Change VM NIC to VirtIO |
| Intermittent connectivity | MTU mismatch | Check MTU along the entire path |
| VM can't reach internet | No default gateway or NAT not configured | Set default gateway, configure NAT if needed |
| Bonding not working | Switch doesn't support LACP | Use active-backup mode instead of 802.3ad |

---

## Quick Reference

```bash
# Proxmox
brctl show                          # View bridges
qm set 100 -net0 virtio,bridge=vmbr0,tag=10   # Set VM NIC with VLAN
ifreload -a                         # Apply /etc/network/interfaces

# OVS
ovs-vsctl show                      # View OVS config
ovs-vsctl add-br br0                # Create bridge
ovs-vsctl add-port br0 ens18        # Add physical NIC

# General Linux networking
ip addr                             # View interfaces
ip route                            # View routes
ip link set eth0 mtu 9000          # Set MTU
tcpdump -i vmbr0 -n                # Capture traffic on bridge

# Hyper-V PowerShell
Get-VMSwitch                        # List switches
New-VMSwitch -Name "name" -SwitchType Internal
Get-VMNetworkAdapter -VMName "VM"   # View VM NICs
Set-VMNetworkAdapterVlan -VMName "VM" -Access -VlanId 10
```

---

## Notes
- Always use **VirtIO** drivers for best performance on KVM/Proxmox — emulated NICs are significantly slower
- The physical switch port must be a **trunk** when using VLAN-aware bridges
- **Bonding mode 4 (LACP)** requires the physical switch to have LACP configured — use mode 1 if unsure
- **Jumbo frames** require all devices in the path to support them — enable on switch, host, AND VM
- OVS is more powerful than Linux bridges but more complex — use Linux bridges for simple setups
- Always capture traffic at the bridge level (`tcpdump -i vmbr0`) when debugging VM connectivity

---

## Related Documents
- [Virtual Machine Routing](virtual-machine-routing.md)
- [Virtual Machine Security Basics](virtual-machine-security-basics.md)
- [VLAN Configuration](../Networking/vlan-configuration-cisco-switch.md)
- [Hyper-V Installation](../Windows/hyperv-installation.md)
