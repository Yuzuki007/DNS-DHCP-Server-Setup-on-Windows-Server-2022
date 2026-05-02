# DNS-DHCP-Server-Setup-on-Windows-Server-2022
This home lab project demonstrates the configuration of DNS (Domain Name System) and DHCP (Dynamic Host Configuration Protocol) on a Windows Server 2022 virtual machine running inside Oracle VirtualBox. The server is also configured as an Active Directory Domain Controller.
This project is part of my IT portfolio to showcase skills in Windows Server administration, networking, and infrastructure setup.

🏗️ Lab Environment
ComponentDetailsHypervisorOracle VirtualBoxOperating SystemWindows Server 2022 (64-bit)RAM4096 MBStorage50 GB (VDI)Network AdapterBridged AdapterServer IP Address192.168.1.200Subnet Mask255.255.255.0Default Gateway192.168.1.1

🎯 Objectives

 Install and verify Active Directory Domain Services (AD DS)
 Install DHCP Server role
 Authorize DHCP Server in Active Directory
 Create and configure a DHCP scope
 Configure DNS forward lookup zone
 Configure DNS reverse lookup zone
 Add DNS A record and CNAME alias
 Verify DNS resolution

<img width="564" height="358" alt="image" src="https://github.com/user-attachments/assets/f62eef89-bebe-43f3-8877-7f7d9f2ef179" />


🔧 DHCP Configuration
Installation
The DHCP Server role was installed using PowerShell:
powershellInstall-WindowsFeature -Name DHCP -IncludeManagementTools
<img width="600" height="158" alt="image" src="https://github.com/user-attachments/assets/88f401ef-1dc7-4663-af66-f02ff1db00aa" />
<img width="433" height="187" alt="image" src="https://github.com/user-attachments/assets/d4f20712-b31b-4be3-9084-324a79660d11" />


Authorization in Active Directory
powershellAdd-DhcpServerInDC -DnsName "WinServer2022" -IPAddress 192.168.1.200
Scope Configuration
SettingValueScope NameHomeLabScopeScope ID192.168.1.0Start Range192.168.1.50End Range192.168.1.150Subnet Mask255.255.255.0Lease Duration8 hoursStateActive ✅
powershellAdd-DhcpServerv4Scope -Name "HomeLabScope" `
  -StartRange 192.168.1.50 `
  -EndRange 192.168.1.150 `
  -SubnetMask 255.255.255.0 `
  -Description "Home Lab DHCP Scope" `
  -State Active
Scope Options
OptionValueRouter/Gateway192.168.1.1DNS Server192.168.1.200DNS DomainWinServer2022
powershellSet-DhcpServerv4OptionValue -ScopeId 192.168.1.0 `
  -Router 192.168.1.1 `
  -DnsServer 192.168.1.200 `
  -DnsDomain "WinServer2022"

  <img width="958" height="315" alt="image" src="https://github.com/user-attachments/assets/f7db4940-c2ec-4162-9619-c2ba19b17573" />


🌐 DNS Configuration
Zones Created
Zone NameZone TypePurposehomelab.localPrimaryForward Lookup Zone1.168.192.in-addr.arpaPrimaryReverse Lookup Zonemylab.localPrimaryAD Integrated Zone
powershell# Forward Lookup Zone
Add-DnsServerPrimaryZone -Name "homelab.local" `
  -ZoneFile "homelab.local.dns" `
  -DynamicUpdate NonSecureAndSecure

# Reverse Lookup Zone
Add-DnsServerPrimaryZone -NetworkId "192.168.1.0/24" `
  -ZoneFile "1.168.192.in-addr.arpa.dns" `
  -DynamicUpdate NonSecureAndSecure
DNS Records
Record TypeNameValueA Recordwinserver2022192.168.1.200CNAMEdcwinserver2022.homelab.local
powershell# A Record with PTR
Add-DnsServerResourceRecordA -Name "winserver2022" `
  -ZoneName "homelab.local" `
  -IPv4Address "192.168.1.200" `
  -CreatePtr

  <img width="411" height="161" alt="image" src="https://github.com/user-attachments/assets/b7ae30bc-5504-401d-a6a0-d2b3ea947904" />


# CNAME Alias
Add-DnsServerResourceRecordCName -Name "dc" `
  -ZoneName "homelab.local" `
  -HostNameAlias "winserver2022.homelab.local."
DNS Resolution Test ✅
powershellResolve-DnsName "winserver2022.homelab.local" -Server 192.168.1.200
Result:
Name     : winserver2022.homelab.local
QueryType: A
TTL      : 3600
Section  : Answer
IPAddress: 192.168.1.200
<img width="355" height="830" alt="image" src="https://github.com/user-attachments/assets/0eee9cf2-b6b9-49b7-9e6f-d5d4f71d8ef9" />

<img width="733" height="343" alt="image" src="https://github.com/user-attachments/assets/e302dfd9-0d7d-4e8e-a3a7-a2103ef6d22a" />



🔍 Verification Commands
powershell# Verify DHCP authorization
Get-DhcpServerInDC

# Verify DHCP scope
Get-DhcpServerv4Scope

# Verify scope options
Get-DhcpServerv4OptionValue -ScopeId 192.168.1.0

# Verify DNS zones
Get-DnsServerZone

# Verify DNS A records
Get-DnsServerResourceRecord -ZoneName "homelab.local" -RRType A

# Test DNS resolution
Resolve-DnsName "winserver2022.homelab.local" -Server 192.168.1.200

📐 Network Diagram
┌─────────────────────────────────────────────┐
│           Oracle VirtualBox Host             │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │     Windows Server 2022 VM           │   │
│  │                                      │   │
│  │  ┌─────────┐  ┌──────┐  ┌────────┐  │   │
│  │  │  AD DS  │  │ DHCP │  │  DNS   │  │   │
│  │  └─────────┘  └──────┘  └────────┘  │   │
│  │                                      │   │
│  │  IP: 192.168.1.200                   │   │
│  │  Domain: homelab.local               │   │
│  └──────────────────────────────────────┘   │
│                                             │
│  DHCP Pool: 192.168.1.50 - 192.168.1.150   │
│  Gateway:   192.168.1.1                     │
└─────────────────────────────────────────────┘
<img width="869" height="689" alt="image" src="https://github.com/user-attachments/assets/33529934-bbbb-489f-adf8-5b74e8ae2069" />


📚 Skills Demonstrated

Windows Server 2022 Administration
PowerShell Scripting & Automation
DHCP Server Installation & Configuration
DNS Server Configuration & Record Management
Active Directory Integration
Network Infrastructure Planning
Server Role Management


🚀 Future Improvements

 Join a Windows 10/11 client VM to the domain
 Configure DHCP reservations for specific devices
 Set up DNS conditional forwarders
 Implement DHCP failover for redundancy
 Add Group Policy Objects (GPOs)
 Configure Remote Desktop Services

