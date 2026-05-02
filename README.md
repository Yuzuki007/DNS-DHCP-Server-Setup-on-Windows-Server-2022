# 🌐 DNS & DHCP Server Setup on Windows Server 2022

![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-blue?style=flat-square&logo=windows)
![DNS](https://img.shields.io/badge/DNS-Configured-teal?style=flat-square)
![DHCP](https://img.shields.io/badge/DHCP-Configured-teal?style=flat-square)
![Active Directory](https://img.shields.io/badge/Active%20Directory-Integrated-orange?style=flat-square)
![PowerShell](https://img.shields.io/badge/PowerShell-Automation-blue?style=flat-square&logo=powershell)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat-square)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-brightgreen?style=flat-square&logo=github)

> **Home Lab Project — Part 1**
> Configuring a fully functional DNS and DHCP server on Windows Server 2022, integrated with Active Directory Domain Services — simulating real enterprise network infrastructure in a VirtualBox home lab.

🔗 **Live Project Page:** [View on GitHub Pages](https://yuzuki007.github.io/DNS-DHCP-Server-Setup-on-Windows-Server-2022/)

---

## 📑 Table of Contents

- [Lab Environment](#lab-environment)
- [Objectives](#objectives)
- [What Are DNS & DHCP?](#what-are-dns--dhcp)
- [DHCP Configuration](#dhcp-configuration)
- [DNS Configuration](#dns-configuration)
- [DNS Records](#dns-records)
- [Verification](#verification)
- [Architecture](#architecture)
- [Issues & Fixes](#issues--fixes)
- [Skills Demonstrated](#skills-demonstrated)
- [Related Projects](#related-projects)

---

## 🏗️ Lab Environment

| Component | Details |
|---|---|
| **Hypervisor** | Oracle VirtualBox |
| **Operating System** | Windows Server 2022 (64-bit) |
| **Base Memory** | 4096 MB |
| **Storage** | 50 GB (VDI — Normal) |
| **Network Adapter** | Bridged — Intel PRO/1000 MT Desktop |
| **Server IP Address** | 192.168.1.200 |
| **Subnet Mask** | 255.255.255.0 |
| **Default Gateway** | 192.168.1.1 |
| **Domain** | mylab.local |

---

## 🎯 Objectives

- [x] Install and verify Active Directory Domain Services (AD DS)
- [x] Install DHCP Server role via PowerShell
- [x] Authorize DHCP Server in Active Directory
- [x] Create and configure a DHCP scope (192.168.1.50–150)
- [x] Set scope options — Gateway, DNS Server, Domain Name
- [x] Configure DNS forward lookup zone (homelab.local)
- [x] Configure DNS reverse lookup zone (192.168.1.0/24)
- [x] Add DNS A record with automatic PTR creation
- [x] Add DNS CNAME alias record
- [x] Verify DNS name resolution
- [x] Export configuration summary to file

---

## 📖 What Are DNS & DHCP?

### DNS — Domain Name System
DNS is the internet's phone book. It translates human-readable names like `winserver2022.homelab.local` into IP addresses like `192.168.1.200` that computers use to communicate. Without DNS, users would need to memorize every IP address on the network.

### DHCP — Dynamic Host Configuration Protocol
DHCP automatically assigns IP addresses and network settings to devices when they join a network — like a hotel front desk handing out room keys. Without DHCP, every device would need to be manually configured.

### DHCP 4-Step Process (DORA)
```
Device          DHCP Server
  │──── Discover ────►│  "Is there a DHCP server?"
  │◄──── Offer ───────│  "Here's 192.168.1.55!"
  │──── Request ─────►│  "I'll take that IP!"
  │◄── Acknowledge ───│  "It's yours for 8 hours!"
```

### Real-World Use
Every company network uses both services. DNS lets employees reach internal apps by name. DHCP ensures laptops, phones, and printers automatically get the correct network settings without manual configuration.

---

## 🔧 DHCP Configuration

### Step 1 — Install DHCP Server Role

```powershell
Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

**Result:**
```
Success: True  |  Restart Needed: No  |  Feature: {DHCP Server, DHCP Server Tools}
```

### Step 2 — Authorize DHCP in Active Directory

```powershell
Add-DhcpServerInDC -DnsName "WinServer2022" -IPAddress 192.168.1.200
```

> **Why authorize?** In an AD environment, DHCP servers must be authorized to prevent rogue DHCP servers from handing out incorrect IP addresses to clients.

### Step 3 — Create DHCP Scope

```powershell
Add-DhcpServerv4Scope -Name "HomeLabScope" `
  -StartRange 192.168.1.50 `
  -EndRange 192.168.1.150 `
  -SubnetMask 255.255.255.0 `
  -Description "Home Lab DHCP Scope" `
  -State Active
```

### Step 4 — Set Scope Options

```powershell
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 `
  -Router 192.168.1.1 `
  -DnsServer 192.168.1.200 `
  -DnsDomain "WinServer2022"
```

### DHCP Scope Summary

| Setting | Value |
|---|---|
| **Scope Name** | HomeLabScope |
| **Scope ID** | 192.168.1.0 |
| **Start Range** | 192.168.1.50 |
| **End Range** | 192.168.1.150 |
| **Subnet Mask** | 255.255.255.0 |
| **Lease Duration** | 8 hours |
| **Router/Gateway** | 192.168.1.1 |
| **DNS Server** | 192.168.1.200 |
| **State** | Active ✅ |

---

## 🌐 DNS Configuration

### Step 1 — Create Forward Lookup Zone

```powershell
Add-DnsServerPrimaryZone -Name "homelab.local" `
  -ZoneFile "homelab.local.dns" `
  -DynamicUpdate NonSecureAndSecure
```

### Step 2 — Create Reverse Lookup Zone

```powershell
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" `
  -ZoneFile "1.168.192.in-addr.arpa.dns" `
  -DynamicUpdate NonSecureAndSecure
```

### Step 3 — Add DNS Records

```powershell
# A Record with automatic PTR
Add-DnsServerResourceRecordA -Name "winserver2022" `
  -ZoneName "homelab.local" `
  -IPv4Address "192.168.1.200" `
  -CreatePtr

# CNAME Alias
Add-DnsServerResourceRecordCName -Name "dc" `
  -ZoneName "homelab.local" `
  -HostNameAlias "winserver2022.homelab.local."
```

---

## 📊 DNS Records

| Type | Name | Value | Zone | TTL |
|---|---|---|---|---|
| **A** | winserver2022 | 192.168.1.200 | homelab.local | 01:00:00 |
| **CNAME** | dc | winserver2022.homelab.local | homelab.local | 01:00:00 |
| **PTR** | 200.1.168.192 | winserver2022.homelab.local | Reverse Zone | 01:00:00 |

### DNS Zones Created

| Zone Name | Zone Type | Purpose |
|---|---|---|
| `homelab.local` | Primary | Forward Lookup |
| `1.168.192.in-addr.arpa` | Primary | Reverse Lookup |
| `mylab.local` | Primary (AD Integrated) | Active Directory |

---

## ✅ Verification

### Verify DHCP

```powershell
# Check DHCP authorization
Get-DhcpServerInDC

# Check scope
Get-DhcpServerv4Scope

# Check scope options
Get-DhcpServerv4OptionValue -ScopeId 192.168.1.0
```

### Verify DNS

```powershell
# List all DNS zones
Get-DnsServerZone

# Check A records
Get-DnsServerResourceRecord -ZoneName "homelab.local" -RRType A

# Test DNS resolution
Resolve-DnsName "winserver2022.homelab.local" -Server 192.168.1.200
```

### DNS Resolution Test Result ✅

```
Name      : winserver2022.homelab.local
QueryType : A
TTL       : 3600
Section   : Answer
IPAddress : 192.168.1.200
```

---

## 📐 Architecture

```
┌─────────────────────────────────────────────────────┐
│                  mylab.local Domain                  │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │           Windows Server 2022 VM             │   │
│  │                                              │   │
│  │  ┌──────────┐  ┌──────────┐  ┌───────────┐  │   │
│  │  │  AD DS   │  │   DHCP   │  │    DNS    │  │   │
│  │  │    DC    │  │  Server  │  │  Server   │  │   │
│  │  └──────────┘  └──────────┘  └───────────┘  │   │
│  │                                              │   │
│  │  IP Address : 192.168.1.200                  │   │
│  │  Domain     : mylab.local / homelab.local    │   │
│  └──────────────────────────────────────────────┘   │
│                                                     │
│  DHCP Pool : 192.168.1.50 — 192.168.1.150          │
│  Gateway   : 192.168.1.1                            │
│  Lease     : 8 hours                                │
└─────────────────────────────────────────────────────┘
```

---

## 🔴 Issues & Fixes

### Issue 1: DHCP Client Gets 169.254.x.x Address
| | |
|---|---|
| **Cause** | Device couldn't reach DHCP server — Windows assigned APIPA address |
| **Fix** | Verify DHCP service is running and scope is active using `Get-DhcpServerv4Scope` |

### Issue 2: DNS Not Resolving Internal Names
| | |
|---|---|
| **Cause** | Client using external/router DNS instead of 192.168.1.200 |
| **Fix** | Ensure DHCP scope option 006 (DNS Server) is set to 192.168.1.200 |

### Issue 3: DHCP Scope Exhausted
| | |
|---|---|
| **Cause** | All 101 IPs in pool (50–150) have been assigned |
| **Fix** | Expand scope range or shorten lease duration. Monitor with `Get-DhcpServerv4Lease` |

### Issue 4: Rogue DHCP Server Conflict
| | |
|---|---|
| **Cause** | Unauthorized DHCP server responding to client requests |
| **Fix** | AD authorization via `Add-DhcpServerInDC` blocks unauthorized servers automatically |

---

## 📚 Skills Demonstrated

- Windows Server 2022 Administration
- PowerShell Scripting & Automation
- DHCP Server Installation & Configuration
- DNS Server Configuration & Record Management
- Active Directory Integration
- Network Infrastructure Planning
- Server Role Management
- Network Troubleshooting
- IT Documentation

---

## 🚀 Future Improvements

- [ ] Join a Windows 10/11 client VM to the domain ✅ *(Done in Part 2)*
- [ ] Configure DHCP reservations for specific devices
- [ ] Set up DNS conditional forwarders
- [ ] Implement DHCP failover for redundancy
- [ ] Add Group Policy Objects (GPOs)
- [ ] Configure Remote Desktop Services

---

## 🔗 Related Projects

| Project | Description | Link |
|---|---|---|
| **Part 1** | DNS & DHCP Server Setup ← You are here | [View](.) |
| **Part 2** | Windows 10 Client & AD Integration | [View](../part2-client-ad/) |
| **Part 3** | Group Policy Objects (GPOs) | Coming Soon |

---

## 👤 Author

**Neil Marvin Simon**
Home Lab Project — Part 1 | Windows Server 2022 | DNS & DHCP | Active Directory
*Built for IT Portfolio purposes*
