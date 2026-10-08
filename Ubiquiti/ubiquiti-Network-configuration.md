# Ubiquiti Configuration (Factory Reset)

**Category:** Networking    
**Last Updated:** 2026-10-07   
**Author:** Brady Genik 

---

## Terminology

| Term | What It Means |
|------|---------------|
| **UniFi** | Ubiquiti's Monitoring Platform |
| **UDM SE** | Ubiquiti firewall used to create a site|
| **U6+** | Wi-Fi 6 wireless access point, mid-range |
| **U6 lite** | Lower-powered Wi-Fi 6 access point, compact form factor |
| **USW-48-G2** | 48 Port Ubiquiti managed gigabit switch |
| **USW Lite 16 PoE** | 16-port managed switch with PoE, small form factor |


---

## Overview
- This Document is used to show up to factory reset each device and re-configure them with new owner/admin
- This exists to help teach future IT technicians on setting up a Network using Ubiquiti devices


---

## Prerequisites
- Access to UniFi Ubiquiti's platform
- [Knowing where everything is located on UniFi](/NHCN/Ubiquiti/unifi-platform.md)


## 1. Restart Firewall (Website)

The Dream Machine firewall is used to create a site on UniFi site manager 

### Physical
1. Use a paper clip or a pin to push down the reset button till screen says factory resetting
2. connect your firewall to the internet and to your laptop via RJ-45/Ethernet
### GUI
3. Open your web browser and it should be up to a device page
4. Answer the following pop up questions:
   - **Name** — [Site]-FW-01 for example BRADY-FW-01
   - **Login** — Login to the Unifi account
   - **Backups** — Unless otherwise told click No Backup
5. Click confirm and it should start to load

> **Tip:** Some info may be incorrect, Will update later - Brady 10/7/2026  

### Images
---
## 2. Adopt Devices

When a site is created you need to add devices to it also known as adopting the devices

### GUI
1. Use RJ-45/Ethernet to connect your adoptable device to the previously connected firewall
2. Open UniFi Site Manager and select the site needed
3. On the left nav bar select the UniFi Devices button
4. Under status you should see "Adopt" on the device you want to adopt

> **Tip:** Some times to adopt the machine you need to hold down the reset button till the indicator light changes 
### Images
![Device Adopting](/Images/Ubiquiti-adopt.png)
---
## 3. Create a Network

Creates a network using internet with a custom Name and Password

### GUI
1. From the same page from section 2 select the settings button on the left nav bar
2. Click the WiFi button and click Create New
   - **Name** — Create a name that makes sense like "Brady Road Office"
   - **Password** —  Create a strong password
   - **Network** — Unless you created another Network select Native Network (Usually ISP's Network)
3. Click the Create button at the bottom of the drop down 
> **Tip:** Remember to document your chosen name and password

### Images
![Network Creation Menu](/Images/Network-Create.png)
---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| STP port blocked | Not Sure Man | DON'T TOUCH IT |

---

## Notes
- Best practices worth remembering
- Things to watch out for
- [UniFi Recovery Mode] (https://help.ui.com/hc/en-us/articles/360043360253-UniFi-Recovery-Mode)


---

## Related Documents
- [Ubiquiti VPN Configurations](/NHCN/Ubiquiti/VPN-configuration-ubiquiti.md)
