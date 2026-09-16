# Windows Server Infrastructure Administration

## Business Scenario

Northstar Solutions is expanding its lab environment from standalone Windows workstation administration to centralized Windows infrastructure.

As the assigned junior IT administrator, I am responsible for deploying and configuring the first Windows Server system, establishing reliable server networking, implementing Active Directory Domain Services and DNS, administering directory objects, integrating the Finance workstation, and preparing the environment for centralized Group Policy.

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
- Inspect and administer Active Directory DNS records
- Configure IPv4 reverse DNS and validate PTR resolution
- Design an Organizational Unit structure for centralized administration
- Create domain-user identities, directory attributes, and manager relationships
- Create and validate departmental Global Security groups
- Practice employee onboarding, transfer, and offboarding administration
- Configure a Windows 11 client to use internal Active Directory DNS
- Join the workstation to the Northstar domain
- Validate Domain Controller discovery and the workstation-domain secure channel
- Verify domain-user authentication and departmental group membership

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

### DNS Record Administration

The existing Active Directory DNS environment was inspected before additional records were created. Inspected structures included `_msdcs`, `_sites`, `_tcp`, `_udp`, `DomainDnsZones`, and `ForestDnsZones`.

Service Location (`SRV`) records were reviewed to understand how clients discover LDAP, Kerberos, Kerberos password services, and the Global Catalog.

An IPv4 reverse lookup zone was created for the Northstar subnet:

| Setting | Value |
|---|---|
| Network | `192.168.252.0/24` |
| Reverse Zone | `252.168.192.in-addr.arpa` |
| Zone Type | Active Directory-integrated primary zone |
| Dynamic Updates | Secure only |
| Domain Controller PTR | `192.168.252.10` → `ns-dc01.ad.northstarsolutions.com` |

Forward and reverse resolution were validated using DNS-only queries explicitly targeting the local DNS service on `NS-DC01`. The retained output shows the expected A and PTR records in the Answer section.

Temporary A and CNAME records were also created, queried, and removed as an administration exercise. They were not retained as permanent infrastructure. MX record concepts were reviewed without creating a fictional mail server.

### Active Directory Organizational Unit Design

A custom OU hierarchy was created for Northstar-managed directory objects:

```text
Northstar
├── Users
│   ├── Finance
│   ├── IT
│   └── Operations
├── Workstations
│   ├── Finance
│   ├── IT
│   └── Operations
├── Groups
├── Service Accounts
└── Disabled Objects
```

The design supports administrative organization, future delegation, Group Policy targeting, and identity lifecycle management. It is not intended merely to reproduce an organizational chart.

The `NS-DC01` computer account remained in the default `Domain Controllers` OU. Accidental-deletion protection was enabled on the custom OUs according to the recorded lab work.

### Domain User Administration

Six fictional employee identities were created in their departmental Users OUs:

| Employee | Account | Department | Role |
|---|---|---|---|
| Ava Chen | `ava.chen` | Finance | Financial Analyst |
| Daniel Kim | `daniel.kim` | Finance | Finance Manager |
| Maya Patel | `maya.patel` | IT | IT Support Technician |
| Ethan Brooks | `ethan.brooks` | IT | Systems Administrator |
| Sofia Martinez | `sofia.martinez` | Operations | Operations Coordinator |
| Lucas Nguyen | `lucas.nguyen` | Operations | Operations Manager |

The accounts were configured with logon identities, title and department attributes, and appropriate OU placement. Initial account setup required a password change at first logon. Temporary passwords are excluded from project documentation.

Manager relationships were configured as:

- Ava Chen → Daniel Kim
- Maya Patel → Ethan Brooks
- Sofia Martinez → Lucas Nguyen

PowerShell verification confirmed the enabled accounts, departmental attributes, job titles, and Distinguished Names. Manager relationships and initial password settings are recorded lab actions; they are not displayed in the retained user-verification screenshot.

### Security Group Administration

Three departmental groups were configured with Global scope and Security category:

| Group | Members |
|---|---|
| `GG-Finance-Users` | Ava Chen, Daniel Kim |
| `GG-IT-Users` | Maya Patel, Ethan Brooks |
| `GG-Operations-Users` | Sofia Martinez, Lucas Nguyen |

This establishes the Accounts → Global Groups portion of AGDLP. Resource-specific Domain Local groups and permissions will be introduced when actual resources such as file shares exist.

Verification identified incorrect initial departmental assignments. The memberships were corrected and re-verified before continuing. This was a configuration-validation correction, not an intentionally induced support incident.

The retained screenshot clearly shows departmental memberships, while a desktop overlay partially obscures the scope/category output. The Global/Security configuration is recorded in the lab notes.

Ordinary IT employee accounts were not added to Domain Admins solely because of their job titles.

### Identity Lifecycle Administration

A separate fictional employee account, `noah.wilson`, was used for onboarding, internal transfer, and offboarding practice.

| Stage | Department / Title | Manager | OU | Departmental Group |
|---|---|---|---|---|
| Onboarding | Finance / Finance Assistant | Daniel Kim | `Northstar > Users > Finance` | `GG-Finance-Users` |
| Internal Transfer | Operations / Operations Analyst | Lucas Nguyen | `Northstar > Users > Operations` | `GG-Operations-Users` |
| Offboarding | Operations attributes retained | Lucas Nguyen | `Northstar > Disabled Objects` | None |

The transfer required separate changes to directory attributes, manager relationship, OU placement, and security-group membership.

For offboarding, the account was disabled, removed from departmental groups, and moved to `Disabled Objects`. The directory object was retained. PowerShell output confirmed `Enabled = False` and only `Domain Users` in the displayed group-membership results.

This was a controlled administration exercise using a fictional identity, not a real employee offboarding or an unexpected outage.

### Windows 11 Domain Integration

Before the join, `NS-W11-01` was recorded as a standalone `WORKGROUP` computer with `CsPartOfDomain = False`.

| Pre-Join Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.128` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.254` |
| DNS Server | `192.168.252.2` |

The workstation could reach `NS-DC01` by IP but could not resolve the Northstar domain or LDAP SRV records through VMware DNS. This was a pre-join validation observation demonstrating the Active Directory DNS dependency.

The workstation's IPv4 DNS server was changed to `192.168.252.10`, while VMware DHCP addressing was retained. The lab notes record successful resolution of the Domain Controller, the domain, LDAP SRV records, and external names after the change.

`NS-W11-01` was then joined to `ad.northstarsolutions.com`. Post-join validation confirmed:

- `CsPartOfDomain = True` and `CsDomain = ad.northstarsolutions.com`.
- Domain Controller discovery locating `NS-DC01` at `192.168.252.10`.
- `Test-ComputerSecureChannel -Verbose` returning `True` in the administrative validation session.
- An enabled computer object placed in `Northstar > Workstations > Finance`.

### Standard Domain User Authentication

A Finance session was validated as `NORTHSTAR\ava.chen` on `NS-W11-01`.

The retained output shows:

- The authenticated domain identity and user SID.
- `NORTHSTAR\GG-Finance-Users` in the user's Windows security token.
- `USERDOMAIN=NORTHSTAR` and `USERDNSDOMAIN=AD.NORTHSTARSOLUTIONS.COM`.
- Successful Domain Controller discovery.
- Local Administrators membership with no individual assignment for Ava Chen; the displayed user token also contains no `BUILTIN\Administrators` entry.

These checks support the documented standard-user session and departmental membership. They do not constitute an audit of all resource permissions. Secure-channel success is documented in the separate administrative validation screenshot.

## Tools and Technologies Used

- Windows Server 2025
- Windows 11 Pro
- VMware Workstation
- Server Manager
- Active Directory Domain Services
- DNS Server
- DNS Manager
- Active Directory Users and Computers
- Active Directory Administrative Tools
- Active Directory PowerShell module
- PowerShell
- Command Prompt
- Windows Server networking
- VMware VMnet8 / NAT
- Windows DNS client configuration
- `nltest`

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

### DNS and Directory Validation on NS-DC01

These commands were used during server-side validation. The loopback DNS address refers to the DNS service on `NS-DC01`.

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Type A -Server 127.0.0.1 -DnsOnly
Resolve-DnsName 192.168.252.10 -Type PTR -Server 127.0.0.1 -DnsOnly
Get-ADUser -SearchBase 'OU=Users,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com' -Filter * -Properties Title,Department | Select-Object Name,SamAccountName,Enabled,Department,Title
Get-ADGroupMember 'GG-Finance-Users' | Select-Object Name,SamAccountName
Get-ADGroupMember 'GG-IT-Users' | Select-Object Name,SamAccountName
Get-ADGroupMember 'GG-Operations-Users' | Select-Object Name,SamAccountName
Get-ADUser noah.wilson -Properties Title,Department | Select-Object Name,Enabled,Title,Department,DistinguishedName
Get-ADPrincipalGroupMembership noah.wilson | Select-Object Name
Get-ADComputer NS-W11-01 -Properties DistinguishedName,Enabled | Select-Object Name,Enabled,DistinguishedName
```

### Domain Integration Validation on NS-W11-01

The domain membership and secure-channel checks were performed in the administrative validation session:

```powershell
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
nltest /dsgetdc:ad.northstarsolutions.com
Test-ComputerSecureChannel -Verbose
```

The domain-user session was inspected separately:

```powershell
whoami
whoami /user
whoami /groups
$env:USERDOMAIN
$env:USERDNSDOMAIN
net localgroup Administrators
nltest /dsgetdc:ad.northstarsolutions.com
```

## Documentation

Supporting technical documentation for this server includes:

- [Windows Server technical baseline and directory configuration](server-baseline.md)
- [Configuration and validation screenshot guide](screenshots/README.md)
- [Windows Server administration knowledge base](../knowledge-base/windows-server-administration.md)
- [Interview preparation based on completed lab work](../interview-prep/windows-server-interview.md)
- [Windows 11 workstation baseline](../01-windows-workstation/system-baseline.md)

This overview incorporates the Phase 4–5 progress recorded through September 16, 2026. The lab notes include actions without separate screenshots, such as temporary record cleanup and manager configuration. Supporting Markdown files are being updated to this checkpoint separately.

Unexpected project incidents and intentionally created real-world support scenarios will be documented separately and labeled accurately as genuine incidents, simulated support incidents, or troubleshooting exercises.

No intentionally induced Phase 4–5 fault scenario is reported complete. The membership correction and pre-join DNS observation are documented as validation findings; the identity lifecycle is a controlled administration exercise.

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
- DNS record inspection and administration
- Reverse lookup zone configuration and PTR validation
- Temporary A and CNAME record practice and cleanup
- Organizational Unit design
- Domain-user attributes and manager relationships
- Departmental Global Security groups and membership validation
- Employee onboarding, transfer, and offboarding exercise
- Windows 11 internal DNS configuration and domain join
- Domain Controller discovery and secure-channel validation
- Active Directory computer-object placement
- Standard domain-user authentication and group-token validation
- Infrastructure documentation

**Current technical checkpoint: Phases 2–5 are complete. Phase 6 — Centralized Group Policy is next and has not yet started hands-on.**

Later stages will include DHCP, file services and permissions, PowerShell automation, server operations, security, monitoring, and integrated troubleshooting scenarios.
