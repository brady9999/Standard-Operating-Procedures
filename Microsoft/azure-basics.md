# Azure Basics
> An introduction to Microsoft Azure — core concepts, services, and common administrative tasks.

**Category:** Microsoft  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Azure** | Microsoft Azure | Microsoft's cloud computing platform |
| **Subscription** | Azure Subscription | A billing container for Azure resources |
| **Resource Group** | Resource Group | A logical container for related Azure resources |
| **Resource** | Azure Resource | Any manageable item in Azure — VM, storage account, database, etc. |
| **Region** | Azure Region | A geographic location where Azure data centers are located |
| **VM** | Virtual Machine | A cloud-based computer running in Azure |
| **VNet** | Virtual Network | An isolated network within Azure |
| **NSG** | Network Security Group | A firewall controlling traffic to Azure resources |
| **Storage Account** | Storage Account | Azure's cloud storage service |
| **Blob** | Binary Large Object | Unstructured data storage in Azure — files, images, backups |
| **IAM** | Identity and Access Management | Controls who can access Azure resources and what they can do |
| **RBAC** | Role-Based Access Control | Assigning roles to users to control Azure access |
| **ARM** | Azure Resource Manager | The deployment and management layer for all Azure resources |
| **Azure CLI** | Azure Command Line Interface | A command-line tool for managing Azure resources |
| **Azure Portal** | Azure Portal | The web-based management interface at portal.azure.com |
| **SKU** | Stock Keeping Unit | The specific size or tier of an Azure resource |
| **SLA** | Service Level Agreement | Microsoft's uptime guarantee for Azure services |
| **HA** | High Availability | Designing systems to minimize downtime |
| **DR** | Disaster Recovery | The ability to recover from a major failure |
| **AKS** | Azure Kubernetes Service | Managed Kubernetes container orchestration |
| **App Service** | Azure App Service | A managed platform for hosting web applications |

---

## Overview
Microsoft Azure is a cloud computing platform offering 200+ services including virtual machines, databases, networking, AI, and security. Azure integrates tightly with Microsoft 365 and on-premises Active Directory through Entra ID (Azure AD).

**Azure Portal:** https://portal.azure.com

**Core concepts hierarchy:**
```
Management Group
└── Subscription
    └── Resource Group
        └── Resources (VMs, Storage, Networking, etc.)
```

---

## 1. Azure Portal Navigation

### Key Areas of the Portal
```
portal.azure.com
├── Home                    ← Dashboard with shortcuts
├── All Services            ← Browse all Azure services
├── Resource Groups         ← View and manage resource groups
├── Subscriptions           ← Billing and quota management
├── Virtual Machines        ← Compute resources
├── Storage Accounts        ← Blob, file, queue, table storage
├── Virtual Networks        ← Networking
├── Azure Active Directory  ← Now called Microsoft Entra ID
├── Monitor                 ← Metrics, logs, alerts
└── Cost Management         ← Billing and cost analysis
```

### Azure CLI Setup
```bash
# Install Azure CLI on Ubuntu
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login

# Set default subscription
az account set --subscription "Subscription Name"

# View current account info
az account show

# List subscriptions
az account list --output table
```

### Azure PowerShell Setup
```powershell
# Install Az module
Install-Module -Name Az -AllowClobber -Force

# Login
Connect-AzAccount

# List subscriptions
Get-AzSubscription

# Set subscription
Set-AzContext -SubscriptionId "subscription-id"
```

---

## 2. Resource Groups

Resource groups are logical containers that hold related Azure resources. All resources must belong to a resource group.

### GUI — Create Resource Group
1. Go to **portal.azure.com**
2. Search for **Resource Groups** → **Create**
3. Select **Subscription**
4. Enter **Resource group name** e.g. `rg-production`
5. Select **Region** e.g. `Canada East`
6. Click **Review + Create** → **Create**

### Azure CLI
```bash
# Create resource group
az group create --name rg-production --location canadaeast

# List resource groups
az group list --output table

# Delete resource group (deletes ALL resources inside)
az group delete --name rg-production --yes --no-wait

# View resources in a group
az resource list --resource-group rg-production --output table
```

### PowerShell
```powershell
# Create resource group
New-AzResourceGroup -Name "rg-production" -Location "canadaeast"

# List resource groups
Get-AzResourceGroup | Select-Object ResourceGroupName, Location

# Delete resource group
Remove-AzResourceGroup -Name "rg-production" -Force
```

---

## 3. Virtual Machines

### GUI — Create a VM
1. Go to **Virtual Machines** → **Create** → **Azure virtual machine**
2. Fill in:
   - **Resource group** — select existing or create new
   - **Virtual machine name** e.g. `vm-webserver-01`
   - **Region** — choose closest to users
   - **Image** — Windows Server 2022, Ubuntu 22.04, etc.
   - **Size** — choose based on CPU/RAM needs
   - **Authentication** — SSH key or password (Linux), password (Windows)
3. **Disks** tab — choose OS disk type (Standard SSD, Premium SSD)
4. **Networking** tab — configure VNet, subnet, public IP, NSG
5. **Management** tab — enable monitoring, auto-shutdown
6. Click **Review + Create** → **Create**

### Azure CLI
```bash
# Create a Linux VM
az vm create \
  --resource-group rg-production \
  --name vm-webserver-01 \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys

# Create a Windows VM
az vm create \
  --resource-group rg-production \
  --name vm-windows-01 \
  --image Win2022Datacenter \
  --size Standard_B2s \
  --admin-username azureadmin \
  --admin-password "StrongPass123!"

# List VMs
az vm list --output table

# Start/stop/restart VM
az vm start --resource-group rg-production --name vm-webserver-01
az vm stop --resource-group rg-production --name vm-webserver-01
az vm restart --resource-group rg-production --name vm-webserver-01

# Deallocate VM (stop billing for compute)
az vm deallocate --resource-group rg-production --name vm-webserver-01

# Delete VM
az vm delete --resource-group rg-production --name vm-webserver-01 --yes

# Open a port in the NSG
az vm open-port --resource-group rg-production --name vm-webserver-01 --port 80
az vm open-port --resource-group rg-production --name vm-webserver-01 --port 443

# Get VM public IP
az vm list-ip-addresses --resource-group rg-production --name vm-webserver-01
```

### Common VM Sizes

| Size | vCPUs | RAM | Use Case |
|------|-------|-----|---------|
| B1s | 1 | 1GB | Dev/test, very light workloads |
| B2s | 2 | 4GB | Light web servers, small databases |
| B4ms | 4 | 16GB | Medium web apps |
| D4s_v5 | 4 | 16GB | General purpose production |
| E4s_v5 | 4 | 32GB | Memory intensive workloads |
| F4s_v2 | 4 | 8GB | CPU intensive workloads |

---

## 4. Virtual Networking

### VNet and Subnet Architecture
```
Virtual Network (10.0.0.0/16)
├── Subnet: web-subnet (10.0.1.0/24)     ← Web servers
├── Subnet: app-subnet (10.0.2.0/24)     ← Application servers
├── Subnet: db-subnet (10.0.3.0/24)      ← Databases (no internet access)
└── Subnet: AzureBastionSubnet           ← Required for Azure Bastion
```

### GUI — Create VNet
1. Search **Virtual Networks** → **Create**
2. Set name, region, and resource group
3. Define address space e.g. `10.0.0.0/16`
4. Add subnets e.g. `10.0.1.0/24`
5. Click **Create**

### Azure CLI
```bash
# Create VNet
az network vnet create \
  --resource-group rg-production \
  --name vnet-main \
  --address-prefix 10.0.0.0/16 \
  --subnet-name web-subnet \
  --subnet-prefix 10.0.1.0/24

# Add a subnet
az network vnet subnet create \
  --resource-group rg-production \
  --vnet-name vnet-main \
  --name db-subnet \
  --address-prefix 10.0.3.0/24

# List VNets
az network vnet list --output table

# List subnets
az network vnet subnet list --resource-group rg-production --vnet-name vnet-main --output table
```

### Network Security Groups (NSG)

NSGs control inbound and outbound traffic to Azure resources — like a firewall.

```bash
# Create NSG
az network nsg create \
  --resource-group rg-production \
  --name nsg-webserver

# Add rule to allow HTTP
az network nsg rule create \
  --resource-group rg-production \
  --nsg-name nsg-webserver \
  --name AllowHTTP \
  --priority 100 \
  --protocol Tcp \
  --destination-port-range 80 \
  --access Allow \
  --direction Inbound

# Add rule to allow HTTPS
az network nsg rule create \
  --resource-group rg-production \
  --nsg-name nsg-webserver \
  --name AllowHTTPS \
  --priority 110 \
  --protocol Tcp \
  --destination-port-range 443 \
  --access Allow \
  --direction Inbound

# Deny all other inbound (lower priority = evaluated last)
az network nsg rule create \
  --resource-group rg-production \
  --nsg-name nsg-webserver \
  --name DenyAllInbound \
  --priority 4096 \
  --protocol "*" \
  --destination-port-range "*" \
  --access Deny \
  --direction Inbound

# List NSG rules
az network nsg rule list \
  --resource-group rg-production \
  --nsg-name nsg-webserver \
  --output table
```

---

## 5. Storage Accounts

### GUI — Create Storage Account
1. Search **Storage Accounts** → **Create**
2. Select resource group
3. Enter **Storage account name** (must be globally unique, lowercase only)
4. Select **Region**
5. Select **Performance**:
   - **Standard** — general purpose (HDD-backed)
   - **Premium** — high performance (SSD-backed)
6. Select **Redundancy**:
   - **LRS** — 3 copies in one datacenter
   - **ZRS** — 3 copies across availability zones
   - **GRS** — copies in two regions
7. Click **Review + Create** → **Create**

### Azure CLI
```bash
# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group rg-production \
  --location canadaeast \
  --sku Standard_LRS \
  --kind StorageV2

# Create a blob container
az storage container create \
  --name mycontainer \
  --account-name mystorageaccount

# Upload a file to blob storage
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./myfile.txt

# List blobs in container
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --output table

# Download a blob
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./downloaded.txt
```

---

## 6. Azure RBAC (Role-Based Access Control)

RBAC controls who can do what with Azure resources.

### Common Built-in Roles

| Role | What It Can Do |
|------|---------------|
| **Owner** | Full access including managing access |
| **Contributor** | Create and manage resources but can't manage access |
| **Reader** | View resources only |
| **Virtual Machine Contributor** | Manage VMs but not the network or storage |
| **Storage Blob Data Contributor** | Read/write/delete blob storage |
| **Network Contributor** | Manage network resources |

### GUI — Assign Role
1. Go to the resource, resource group, or subscription
2. Click **Access Control (IAM)** in the left menu
3. Click **Add** → **Add role assignment**
4. Select a **Role**
5. Select **Members** → search for user/group
6. Click **Review + Assign**

### Azure CLI
```bash
# Assign a role to a user at resource group scope
az role assignment create \
  --assignee user@company.com \
  --role "Contributor" \
  --resource-group rg-production

# Assign role at subscription scope
az role assignment create \
  --assignee user@company.com \
  --role "Reader" \
  --scope /subscriptions/{subscription-id}

# List role assignments
az role assignment list --resource-group rg-production --output table

# Remove a role assignment
az role assignment delete \
  --assignee user@company.com \
  --role "Contributor" \
  --resource-group rg-production
```

---

## 7. Azure Monitor and Alerts

```bash
# View VM metrics via CLI
az monitor metrics list \
  --resource "/subscriptions/{sub-id}/resourceGroups/rg-production/providers/Microsoft.Compute/virtualMachines/vm-webserver-01" \
  --metric "Percentage CPU" \
  --output table

# Create a metric alert
az monitor metrics alert create \
  --name "High CPU Alert" \
  --resource-group rg-production \
  --scopes "/subscriptions/{sub-id}/resourceGroups/rg-production/providers/Microsoft.Compute/virtualMachines/vm-webserver-01" \
  --condition "avg Percentage CPU > 90" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --severity 2
```

---

## 8. Cost Management

```bash
# View current costs
az consumption usage list --output table

# View budget
az consumption budget list --output table

# Set spending alert via portal:
# Cost Management → Budgets → Add budget → Set alerts at 80% and 100%
```

**Cost saving tips:**
- Use **B-series VMs** for dev/test — they're much cheaper
- **Deallocate VMs** when not in use — you only pay for storage, not compute
- Use **Azure Reservations** for 1-3 year commitments — up to 72% savings
- Set up **budget alerts** to avoid bill surprises
- Delete unused resources — **Resource Groups**, **disks**, **public IPs** all cost money even when idle

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| Can't create resource | No permissions | Check IAM role assignments |
| VM can't be reached | NSG blocking port | Add inbound NSG rule for the port |
| Storage access denied | Firewall or RBAC | Check storage account firewall and role assignments |
| High unexpected costs | Resources left running | Review Cost Management, deallocate unused VMs |
| Login fails | Wrong subscription | Run `az account set` to switch subscription |
| VNet peering not working | Overlapping address spaces | Use non-overlapping CIDR ranges |

---

## Quick Reference

```bash
# Login and setup
az login
az account set --subscription "name"

# Resource groups
az group create --name rg-name --location canadaeast
az group list --output table

# VMs
az vm create --resource-group rg --name vm-name --image Ubuntu2204
az vm start/stop/restart --resource-group rg --name vm-name
az vm list --output table

# Storage
az storage account create --name name --resource-group rg --sku Standard_LRS

# RBAC
az role assignment create --assignee user@domain.com --role "Contributor" --resource-group rg

# Monitor
az monitor metrics list --resource "/resource/id" --metric "Percentage CPU"
```

---

## Notes
- All Azure resources must belong to a **Resource Group** — plan your resource group structure before creating resources
- **Regions** affect latency, compliance, and pricing — choose the region closest to your users
- **NSG rules** are processed by priority — lower number = higher priority — first match wins
- Always use **service principals** for automated deployments — never use personal credentials in scripts
- **Azure Cost Management** is essential — cloud costs can grow unexpectedly without monitoring
- The **Azure Well-Architected Framework** covers reliability, security, cost, performance, and operations — worth reading

---

## Related Documents
- [Entra ID Basics](entra-id-basics.md)
- [Intune Basics](intune-basics.md)
- [Microsoft 365 User Management](microsoft-365-user-management.md)
