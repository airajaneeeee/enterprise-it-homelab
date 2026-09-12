# Windows Server Infrastructure Administration

## Business Scenario

Northstar Solutions is expanding its lab environment from standalone Windows workstation administration to centralized Windows infrastructure.

As the assigned junior IT administrator, I am responsible for deploying and configuring the first Windows Server system, establishing reliable server networking, implementing Active Directory Domain Services and DNS, validating the new domain infrastructure, and preparing the environment for centralized user, computer, and policy management.

## Server

- Computer Name: `NS-DC01`
- Fully Qualified Domain Name: `NS-DC01.ad.northstarsolutions.com`
- Operating System: Windows Server 2025 Standard Evaluation
- Platform: VMware Workstation
- Role: Domain Controller / DNS Server
- Active Directory Domain: `ad.northstarsolutions.com`

## Lab Objectives

- Deploy and configure Windows Server 2025
- Establish a standardized server identity
- Assess the VMware virtual network
- Configure predictable static IPv4 addressing
- Validate network connectivity before deploying infrastructure services
- Install Active Directory Domain Services
- Install DNS Server
- Create a new Active Directory forest and domain
- Promote `NS-DC01` to the first Domain Controller
- Configure Global Catalog services
- Validate Active Directory domain and forest configuration
- Validate Active Directory and DNS service status
- Verify internal and external DNS resolution
- Prepare the environment for centralized identity and workstation administration
- Document infrastructure configuration and validation evidence

## Completed Work

### Windows Server Deployment

- Created a dedicated Windows Server 2025 virtual machine in VMware Workstation.
- Configured 2 virtual processors, 4 GB RAM, approximately 60 GB of virtual storage, and VMware NAT networking.
- Renamed the Windows Server system to `NS-DC01` using the Northstar Solutions naming convention.
- Verified the server hostname after restart.
- Established a technical baseline before introducing infrastructure roles.

### Network Assessment and Static IPv4 Configuration

- Inspected the VMware VMnet8 NAT network before assigning infrastructure addressing.
- Identified the `192.168.252.0/24` network.
- Identified the VMware NAT gateway at `192.168.252.2`.
- Identified the VMware DHCP allocation range of `192.168.252.128-192.168.252.254`.
- Confirmed that `NS-DC01` initially received `192.168.252.129` through DHCP.
- Reconfigured the server with the static IPv4 address `192.168.252.10`.
- Configured subnet mask `255.255.255.0` and default gateway `192.168.252.2`.
- Disabled DHCP addressing on the server.
- Configured the server to use itself for DNS in preparation for hosting the Northstar internal DNS service.
- After Domain Controller promotion, verified the current IPv4 DNS client setting as `127.0.0.1`, which points back to the DNS service running locally on `NS-DC01`.

### Pre-Deployment Network and DNS Validation

- Verified connectivity to the VMware NAT gateway.
- Verified external IP connectivity using `ping 8.8.8.8`.
- Tested DNS resolution separately from general network connectivity.
- Established that external IP connectivity worked while DNS resolution failed before the DNS Server role was installed.
- Used the pre-deployment result as a baseline for comparison after DNS deployment.

### Active Directory Domain Services and DNS Deployment

- Installed the Active Directory Domain Services (AD DS) role.
- Installed the DNS Server role.
- Promoted `NS-DC01` as the first Domain Controller in a new Active Directory forest.
- Created the Active Directory forest and root domain:

```text
ad.northstarsolutions.com
```

- Configured the NetBIOS domain name:

```text
NORTHSTAR
```

- Used Windows Server 2025 forest and domain functional levels.
- Configured `NS-DC01` as a DNS Server and Global Catalog.
- Completed Domain Controller promotion and restarted the server into the new domain environment.

### Active Directory Validation

- Verified the administrative security context changed to the Northstar domain.
- Queried the Active Directory domain using PowerShell.
- Queried the Active Directory forest using PowerShell.
- Confirmed `ad.northstarsolutions.com` as the domain and forest root.
- Confirmed `NORTHSTAR` as the NetBIOS domain name.
- Confirmed `NS-DC01.ad.northstarsolutions.com` as the Domain Controller and Global Catalog.
- Verified the Active Directory Domain Services (`NTDS`) service was running.
- Verified the DNS Server (`DNS`) service was running.

### DNS Validation

- Tested resolution of the internal Domain Controller FQDN:

```text
NS-DC01.ad.northstarsolutions.com
```

- Verified that the internal name resolved to:

```text
192.168.252.10
```

- Tested external DNS resolution using `microsoft.com`.
- Verified successful external name resolution after DNS Server deployment.
- Confirmed that `NS-DC01` could provide internal Active Directory name resolution while also resolving external DNS names.
- Verified that no explicit DNS forwarders are currently configured.
- Confirmed that external DNS resolution still succeeds using the Windows DNS server's root-hints capability.
- Observed that `nslookup` could select the IPv6 loopback address `::1` when querying the local DNS service.
- Kept IPv6 enabled rather than disabling it solely because the local DNS server was represented by `::1`.

### Post-Deployment Server Baseline

After AD DS and DNS deployment, the server administration baseline was reviewed before beginning Active Directory object administration.

- Verified the active network profile is `DomainAuthenticated`.
- Verified IPv4 Internet connectivity.
- Kept IPv6 enabled; the baseline check showed no active IPv6 traffic.
- Verified Remote Desktop is enabled.
- Kept Network Level Authentication enabled for Remote Desktop.
- Verified Windows Remote Management is enabled.
- Verified the server time zone.
- Verified Windows Update settings/status.
- Verified VMware Tools is installed and operational.
- Kept Windows Defender Firewall enabled.
- Enabled the inbound `File and Printer Sharing (Echo Request - ICMPv4-In)` rule to support lab connectivity testing and troubleshooting.
- Reviewed DNS forwarder configuration and confirmed that no explicit forwarders are currently configured.

## Tools and Technologies Used

- Windows Server 2025
- VMware Workstation
- Server Manager
- Active Directory Domain Services
- DNS Server
- Active Directory Users and Computers
- Active Directory Administrative Tools
- PowerShell
- Command Prompt
- Windows Server networking
- VMware VMnet8 / NAT

## Commands Used

### Command Prompt

```cmd
hostname
ipconfig /all
ping 192.168.252.2
ping 8.8.8.8
nslookup microsoft.com
```

### PowerShell

```powershell
hostname
whoami
Get-ADDomain
Get-ADForest
Get-Service NTDS,DNS
Resolve-DnsName NS-DC01.ad.northstarsolutions.com
Resolve-DnsName microsoft.com
```

## Documentation

Supporting technical documentation for this server includes:

- Windows Server technical baseline
- Active Directory and DNS configuration
- Configuration and validation screenshots
- Windows Server administration knowledge base
- Interview preparation based on completed lab work

Unexpected project incidents and intentionally created real-world support scenarios will be documented separately and labeled accurately as genuine incidents, simulated support incidents, or troubleshooting exercises.

## Production Considerations

This homelab applies production-oriented infrastructure practices while adapting them to a resource-constrained single-host virtual environment.

A production Active Directory implementation would additionally consider:

- Multiple Domain Controllers for redundancy and availability
- Redundant DNS services
- Formal IP address management
- Dedicated server networks or VLANs
- Privileged administrative account separation
- Centralized monitoring and logging
- Active Directory backup and recovery
- Security baselines and auditing
- Disaster recovery planning
- Formal change management

The single-Domain-Controller design used in this homelab is appropriate for learning and functional validation but does not provide production-level redundancy.

## Module Status

**Windows Server Infrastructure Administration — In Progress**

Completed areas:

- Windows Server 2025 deployment
- Server identity standardization
- VMware virtual network assessment
- Static IPv4 infrastructure addressing
- Network connectivity validation
- Active Directory Domain Services installation
- DNS Server installation
- New Active Directory forest deployment
- Domain Controller promotion
- Global Catalog configuration
- Active Directory domain and forest validation
- Active Directory and DNS service validation
- Internal DNS resolution validation
- External DNS resolution validation
- Domain-authenticated network profile validation
- Remote Desktop validation
- Network Level Authentication validation
- Remote Management validation
- Windows Firewall / ICMPv4 troubleshooting rule configuration
- DNS forwarding/root-hints review
- IPv6 retained and reviewed rather than disabled
- VMware Tools verification
- Time zone verification
- Windows Update baseline check
- Infrastructure documentation

The server infrastructure will continue to be developed with Organizational Units, domain users, security groups, Windows 11 domain integration, centralized Group Policy, DHCP, file services, PowerShell administration, security configuration, and infrastructure troubleshooting.
