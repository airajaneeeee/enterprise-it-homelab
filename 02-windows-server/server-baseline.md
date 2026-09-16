# System Baseline — NS-DC01

## Purpose

This document records the technical baseline and infrastructure configuration of `NS-DC01`, the Windows Server 2025 system used in the Northstar Solutions enterprise IT homelab.

The baseline documents the server's identity, virtual hardware, network configuration, pre-deployment state, Active Directory Domain Services configuration, DNS configuration, directory administration, and domain-client integration.

The current administration checkpoint reflects the supplied lab notes and retained evidence through September 16, 2026. Earlier deployment observations remain historical records. Actions reported in the notes are distinguished from settings directly visible in the selected screenshots.

## System Identity

| Item | Value |
|---|---|
| VMware VM Name | `Northstar-DC01` |
| Windows Hostname | `NS-DC01` |
| FQDN | `NS-DC01.ad.northstarsolutions.com` |
| Operating System | Windows Server 2025 Standard Evaluation |
| Windows Version | 2009 |
| OS Build | 26100 |
| Platform | VMware Workstation |
| Current Role | Domain Controller / DNS Server |
| Active Directory Domain | `ad.northstarsolutions.com` |
| NetBIOS Domain | `NORTHSTAR` |

## Virtual Hardware

| Component | Configuration |
|---|---|
| Processor | 2 vCPU |
| Memory | 4 GB |
| Virtual Disk | ~60 GB |
| Network Mode | VMware NAT / VMnet8 |

The virtual machine is intentionally resource-constrained for homelab use.

Production Domain Controllers would be sized according to workload, directory size, authentication demand, monitoring requirements, redundancy design, and organizational standards.

## VMware Network

| Setting | Value |
|---|---|
| Virtual Network | VMnet8 |
| Network Address | `192.168.252.0/24` |
| Subnet Mask | `255.255.255.0` |
| NAT / Default Gateway | `192.168.252.2` |
| DHCP Range | `192.168.252.128-192.168.252.254` |

## Initial Network Configuration

Before infrastructure services were deployed, `NS-DC01` received its network configuration through VMware DHCP.

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.129` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.254` |
| DNS Server | `192.168.252.2` |
| DNS Suffix | `localdomain` |
| DHCP Enabled | Yes |

The initial configuration was recorded before assigning the server a predictable infrastructure address.

## Static Network Configuration

`NS-DC01` was reconfigured with static IPv4 addressing before Active Directory and DNS deployment.

The table below records the later DNS client setting after Domain Controller promotion. During the original static-address configuration, the preferred DNS server was `192.168.252.10`; that earlier value remains visible in the historical screenshot and pre-deployment test output.

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.10` |
| Subnet Mask | `255.255.255.0` |
| Prefix Length | `/24` |
| Default Gateway | `192.168.252.2` |
| Preferred DNS | `127.0.0.1` |
| Alternate DNS | None |
| DHCP Enabled | No |

`NS-DC01` uses its own local DNS service because it hosts DNS for the Northstar Solutions Active Directory domain.

The current IPv4 DNS client setting is `127.0.0.1`, which is the IPv4 loopback address and points back to the DNS service running on the same server.

The server itself remains reachable on the network at:

```text
192.168.252.10
```

## Pre-Deployment Validation

Network connectivity and DNS behavior were tested before Active Directory Domain Services and DNS Server were deployed.

### Hostname

```cmd
hostname
```

Verified result:

```text
NS-DC01
```

### Gateway Connectivity

```cmd
ping 192.168.252.2
```

**Result:** Successful

### External IP Connectivity

```cmd
ping 8.8.8.8
```

**Result:**

- Sent: 4
- Received: 4
- Lost: 0
- Packet loss: 0%

### Pre-DNS Resolution Test

```cmd
nslookup microsoft.com
```

Observed result:

```text
Server:  UnKnown
Address: 192.168.252.10

*** UnKnown can't find microsoft.com: No response from server
```

At this stage, `NS-DC01` was configured to query its own address for DNS, but the DNS Server role had not yet been installed.

This established a pre-deployment DNS baseline.

## Active Directory Domain Services Configuration

The following Windows Server roles were installed:

- Active Directory Domain Services
- DNS Server

`NS-DC01` was then promoted as the first Domain Controller in a new Active Directory forest.

### Active Directory Configuration

| Item | Value |
|---|---|
| Forest Root Domain | `ad.northstarsolutions.com` |
| Active Directory Domain | `ad.northstarsolutions.com` |
| Distinguished Name | `DC=ad,DC=northstarsolutions,DC=com` |
| NetBIOS Domain | `NORTHSTAR` |
| Forest Functional Level | Windows Server 2025 |
| Domain Functional Level | Windows Server 2025 |
| Domain Controller | `NS-DC01.ad.northstarsolutions.com` |
| DNS Server | Enabled |
| Global Catalog | Enabled |
| Read-Only Domain Controller | No |

The default Active Directory database, log, and SYSVOL paths were used for this homelab deployment.

A Directory Services Restore Mode (DSRM) password was configured during promotion but is intentionally not recorded in project documentation.

## Domain Controller Validation

Post-deployment validation was performed after Domain Controller promotion and restart.

### Server Identity

```powershell
hostname
whoami
```

Verified results included:

```text
NS-DC01
NORTHSTAR\Administrator
```

This confirmed the server hostname and administrative domain context after promotion.

### Active Directory Domain

The domain configuration was inspected using:

```powershell
Get-ADDomain
```

Verified configuration included:

```text
DNSRoot               : ad.northstarsolutions.com
DistinguishedName     : DC=ad,DC=northstarsolutions,DC=com
DomainMode            : Windows2025Domain
Forest                : ad.northstarsolutions.com
Name                  : ad
NetBIOSName           : NORTHSTAR
PDCEmulator           : NS-DC01.ad.northstarsolutions.com
InfrastructureMaster  : NS-DC01.ad.northstarsolutions.com
RIDMaster             : NS-DC01.ad.northstarsolutions.com
```

### Active Directory Forest

The forest configuration was inspected using:

```powershell
Get-ADForest
```

Verified configuration included:

```text
Domains              : {ad.northstarsolutions.com}
ForestMode           : Windows2025Forest
GlobalCatalogs       : {NS-DC01.ad.northstarsolutions.com}
Name                 : ad.northstarsolutions.com
RootDomain           : ad.northstarsolutions.com
DomainNamingMaster   : NS-DC01.ad.northstarsolutions.com
SchemaMaster         : NS-DC01.ad.northstarsolutions.com
```

The server is currently located in the default Active Directory site:

```text
Default-First-Site-Name
```

## Active Directory and DNS Services

Service status was verified using:

```powershell
Get-Service NTDS,DNS
```

Verified state:

| Service | Function | Status |
|---|---|---|
| `NTDS` | Active Directory Domain Services | Running |
| `DNS` | DNS Server | Running |

This confirmed that both core services were operational after Domain Controller promotion.

## DNS Validation

DNS resolution was retested after Active Directory and DNS deployment.

### Internal DNS Resolution

```powershell
Resolve-DnsName NS-DC01.ad.northstarsolutions.com
```

The internal Domain Controller name successfully resolved to:

```text
192.168.252.10
```

This confirmed internal DNS resolution for the Northstar Solutions Active Directory namespace.

### External DNS Resolution

```powershell
Resolve-DnsName microsoft.com
```

The query successfully returned public IPv4 addresses for `microsoft.com`.

This confirmed that external DNS resolution was functioning through the DNS configuration on `NS-DC01`.

### DNS Forwarders

The DNS Server properties were reviewed.

No explicit DNS forwarders are currently configured.

External DNS resolution still succeeds because Windows DNS can use root hints when no forwarder is configured.

### IPv6 and Local DNS

During `nslookup` testing, the DNS server was displayed as:

```text
::1
```

`::1` is the IPv6 loopback address and refers to the local computer.

IPv6 was intentionally left enabled.

The presence of `::1` alone was not treated as a reason to disable IPv6.

### Before and After Comparison

| Test | Before DNS Deployment | After DNS Deployment |
|---|---|---|
| Gateway connectivity | Successful | Available |
| External IP connectivity | Successful | Available |
| Query DNS server at `192.168.252.10` | No response | Operational |
| Internal AD DNS resolution | Not available | Successful |
| External DNS resolution | Failed through `192.168.252.10` | Successful |

The comparison demonstrates the change from a server configured to query a DNS service that had not yet been deployed to an operational Active Directory DNS server capable of resolving both the internal domain namespace and external DNS names.

## DNS Record Administration State

The existing Active Directory DNS environment was inspected before additional records were created during Phase 4.

### Forward DNS and Service Discovery

The primary Active Directory namespace is:

```text
ad.northstarsolutions.com
```

The lab notes record inspection of the Domain Controller A record and AD-created DNS structures including `_msdcs`, `_sites`, `_tcp`, `_udp`, `DomainDnsZones`, and `ForestDnsZones`. SRV records for LDAP, Kerberos, Kerberos password operations, and the Global Catalog were reviewed.

### Reverse Lookup Zone

| Item | Recorded State |
|---|---|
| Network | `192.168.252.0/24` |
| Reverse Zone | `252.168.192.in-addr.arpa` |
| Zone Type | Primary |
| Active Directory-integrated | Yes |
| Dynamic Updates | Secure only |
| PTR Owner Name | `10.252.168.192.in-addr.arpa` |
| PTR Target | `ns-dc01.ad.northstarsolutions.com` |

The zone configuration is recorded in the lab notes. The retained DNS screenshot shows the A and PTR query results; it does not display the zone's integration or update-policy properties.

### Forward and Reverse Validation

Queries were run on `NS-DC01`, explicitly targeting its local DNS service:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Type A -Server 127.0.0.1 -DnsOnly
Resolve-DnsName 192.168.252.10 -Type PTR -Server 127.0.0.1 -DnsOnly
```

| Query | Returned Value | Displayed Section |
|---|---|---|
| A — `ns-dc01.ad.northstarsolutions.com` | `192.168.252.10` | Answer |
| PTR — `10.252.168.192.in-addr.arpa` | `ns-dc01.ad.northstarsolutions.com` | Answer |

The same screenshot also shows ordinary queries displaying `Section = Question`. This difference was retained as a validation observation. The Answer section alone is not treated as proof of the DNS authoritative-answer flag or as an explanation of how the ordinary queries obtained their results.

See [forward and reverse DNS evidence](screenshots/07-dns-forward-reverse-resolution-verification.png).

### Temporary Record Lifecycle

The notes record creation and validation of a temporary A record, `dns-lab.ad.northstarsolutions.com`, pointing to `192.168.252.10`, and a CNAME, `dns-alias.ad.northstarsolutions.com`, pointing to that temporary hostname.

Both training records were removed after validation. They are not part of the permanent DNS baseline. Creation and cleanup were documented without separate screenshots.

## Post-Deployment Administration Baseline

The following baseline checks were completed after AD DS and DNS deployment and before beginning Active Directory object administration.

| Item | Verified State |
|---|---|
| Network Profile | `DomainAuthenticated` |
| IPv4 Connectivity | Internet |
| IPv6 | Enabled |
| IPv6 Connectivity | `NoTraffic` observed during baseline check |
| Remote Desktop | Enabled |
| Network Level Authentication | Enabled |
| Remote Management | Enabled |
| Windows Defender Firewall | Enabled |
| ICMPv4 Echo Request | Inbound troubleshooting rule enabled |
| DNS Forwarders | None configured |
| External DNS Resolution | Successful |
| External Resolution Method | Root hints available |
| VMware Tools | Installed / operational |
| Time Zone | Verified |
| Windows Update | Verified |

The original notes record completion of the time-zone and Windows Update checks but do not retain the exact time-zone identifier or update result in this baseline.

The Windows Firewall remains enabled.

Rather than disabling the firewall, an ICMPv4 Echo Request inbound rule was enabled to support controlled connectivity testing and troubleshooting within the lab.

IPv6 was also kept enabled rather than disabled simply because the current Northstar environment primarily uses IPv4.

## Current Active Directory Administrative Structure

### Organizational Units

Northstar-managed directory objects are organized under a custom top-level OU:

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

`NS-DC01` remains in the default `Domain Controllers` OU. The notes record accidental-deletion protection on the custom OUs; the [OU screenshot](screenshots/08-active-directory-ou-structure.png) demonstrates the hierarchy and DC placement, not that protection setting.

### Departmental Users

The six active fictional employee identities are:

| Name | Account | Department / Title | Users OU | Manager |
|---|---|---|---|---|
| Ava Chen | `ava.chen` | Finance / Financial Analyst | Finance | Daniel Kim |
| Daniel Kim | `daniel.kim` | Finance / Finance Manager | Finance | Not recorded |
| Maya Patel | `maya.patel` | IT / IT Support Technician | IT | Ethan Brooks |
| Ethan Brooks | `ethan.brooks` | IT / Systems Administrator | IT | Not recorded |
| Sofia Martinez | `sofia.martinez` | Operations / Operations Coordinator | Operations | Lucas Nguyen |
| Lucas Nguyen | `lucas.nguyen` | Operations / Operations Manager | Operations | Not recorded |

All departmental Users OUs in this table are beneath `Northstar > Users`. PowerShell output in the [user-verification screenshot](screenshots/09-active-directory-domain-users-verification.png) shows enabled accounts, account names, departments, titles, and Distinguished Names. Manager relationships are recorded in the lab notes but are not shown in that output. “Not recorded” does not assert that the directory attribute is empty.

The notes also record initial password-change-at-first-logon configuration. Temporary passwords are excluded from documentation, and that initial setting is not asserted as the current state after users have signed in.

### Departmental Security Groups

| Group | Recorded Scope / Category | Members |
|---|---|---|
| `GG-Finance-Users` | Global / Security | Ava Chen, Daniel Kim |
| `GG-IT-Users` | Global / Security | Maya Patel, Ethan Brooks |
| `GG-Operations-Users` | Global / Security | Sofia Martinez, Lucas Nguyen |

The groups implement Accounts → Global Groups from AGDLP. Resource-specific Domain Local groups and permissions have not yet been implemented.

Initial membership assignments were corrected during validation and then re-verified. The [group screenshot](screenshots/10-active-directory-security-groups-verification.png) clearly displays the departmental memberships; a desktop overlay partially obscures the scope/category output. Global/Security configuration is recorded in the notes.

The notes state that the ordinary IT employee accounts were not added to Domain Admins merely because of their job titles.

### Identity Lifecycle and Disabled Objects

The fictional account `noah.wilson` was used for an administration exercise:

| Stage | Title / Department | Manager | Location | Departmental Membership |
|---|---|---|---|---|
| Onboarding | Finance Assistant / Finance | Daniel Kim | `Northstar > Users > Finance` | `GG-Finance-Users` |
| Transfer | Operations Analyst / Operations | Lucas Nguyen | `Northstar > Users > Operations` | `GG-Operations-Users` |
| Offboarding | Operations attributes retained | Lucas Nguyen | `Northstar > Disabled Objects` | None |

Final recorded state:

```text
Name: Noah Wilson
SamAccountName: noah.wilson
Enabled: False
DistinguishedName: CN=Noah Wilson,OU=Disabled Objects,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
Displayed group membership: Domain Users
```

The [onboarding and transfer evidence](screenshots/11-active-directory-identity-lifecycle-offboarding-1.png) shows changes to title, department, OU, and group membership. The [offboarding evidence](screenshots/11-active-directory-identity-lifecycle-offboarding-2.png) shows the retained disabled object and removal of departmental memberships. Manager changes are recorded in the notes rather than the selected output.

This is a controlled identity-lifecycle exercise. The membership correction is a configuration-validation finding. No intentionally induced Phase 4 fault scenario is reported complete.

## Infrastructure Role

`NS-DC01` currently provides:

- Active Directory Domain Services
- Domain authentication infrastructure
- Internal DNS and Active Directory service discovery
- Global Catalog services
- Central directory services for users, groups, and computers

Current architecture:

```text
Northstar Solutions
└── ad.northstarsolutions.com
    ├── NS-DC01
    │   ├── Windows Server 2025
    │   ├── Domain Controller / AD DS
    │   ├── DNS Server / Global Catalog
    │   └── 192.168.252.10
    │
    └── NS-W11-01
        ├── Windows 11 Pro Finance workstation
        ├── Domain joined; VMware DHCP IPv4
        └── Internal DNS → 192.168.252.10
```

## Domain Client Integration State

### Network Configuration Evolution

Before the domain join, the workstation was recorded as `WORKGROUP` with `CsPartOfDomain = False`.

| Setting | Pre-Join Observation | Domain Integration Checkpoint |
|---|---|---|
| Computer | `NS-W11-01` | `NS-W11-01` |
| Address Assignment | VMware DHCP | VMware DHCP retained |
| Observed IPv4 Address | `192.168.252.128` | No static address assigned; DHCP remains in use |
| Subnet Mask | `255.255.255.0` | No change reported |
| Default Gateway | `192.168.252.2` | No change reported |
| DHCP Server | `192.168.252.254` | VMware DHCP retained |
| IPv4 DNS Server | `192.168.252.2` | `192.168.252.10` |

The pre-join address is a recorded DHCP observation, not a permanent address assignment. The workstation queries the DC's network address for DNS; the DC's loopback setting `127.0.0.1` is not used as the workstation's DNS server.

The notes record successful IP connectivity to the DC but failed Northstar domain and LDAP SRV resolution through VMware DNS. After configuring internal DNS, the workstation resolved the DC FQDN, domain, LDAP SRV records, and external names. This was a prerequisite validation observation, not a deliberately induced domain-join failure.

### Domain Membership and Trust

Validation on `NS-W11-01` used:

```powershell
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
nltest /dsgetdc:ad.northstarsolutions.com
Test-ComputerSecureChannel -Verbose
```

| Check | Observed Result |
|---|---|
| Computer | `NS-W11-01` |
| Domain | `ad.northstarsolutions.com` |
| Part of Domain | `True` |
| Discovered DC | `NS-DC01.ad.northstarsolutions.com` |
| DC Address | `192.168.252.10` |
| Secure Channel | `True`; reported in good condition |

These results are visible in the [administrative domain-membership validation](screenshots/12-ns-w11-01-domain-membership-verification.png).

### Active Directory Computer Object

The enabled workstation computer object was placed at:

```text
CN=NS-W11-01,OU=Finance,OU=Workstations,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
```

Server-side PowerShell validation used:

```powershell
Get-ADComputer NS-W11-01 -Properties DistinguishedName,Enabled | Select-Object Name,Enabled,DistinguishedName
```

The [computer-object evidence](screenshots/13-ns-w11-01-ad-computer-object-verification.png) shows `Enabled = True` and the Finance Workstations OU placement. This prepares the client for future centralized Group Policy; it does not establish that a new domain GPO has already been deployed.

### Standard Domain-User Session

The [Finance session evidence](screenshots/14-domain-user-login-and-group-membership.png) shows:

- Authenticated identity `NORTHSTAR\ava.chen`.
- `NORTHSTAR\GG-Finance-Users` in the Windows security token.
- `USERDOMAIN=NORTHSTAR` and `USERDNSDOMAIN=AD.NORTHSTARSOLUTIONS.COM`.
- Successful Domain Controller discovery.
- No individual Ava Chen entry in local Administrators and no `BUILTIN\Administrators` entry in the displayed user token.

The output supports the standard-user session and departmental group membership, not a complete audit of all resource access. The secure-channel command at the bottom of this screenshot has no visible result; the successful result is retained in screenshot 12 from the administrative session.

## Production Considerations

The current configuration is designed for a resource-constrained enterprise-style homelab.

A production Active Directory environment would typically evaluate requirements for:

- Multiple Domain Controllers
- DNS redundancy
- Server and service monitoring
- Active Directory backup and recovery
- Privileged administrative account separation
- Security baselines
- Centralized logging and auditing
- Network segmentation
- Formal IP address management
- Disaster recovery
- Change management

The single-Domain-Controller configuration used here does not provide infrastructure redundancy and should not be interpreted as a production high-availability design.

## Baseline Status

**Windows Server and Active Directory Baseline — Complete through Domain Client Integration**

The documented infrastructure baseline currently includes:

- Windows Server 2025 deployment
- Standardized server identity
- VMware network assessment
- Static IPv4 infrastructure addressing
- Pre-deployment connectivity and DNS assessment
- Active Directory Domain Services installation
- DNS Server installation
- New Active Directory forest and domain
- Domain Controller promotion
- Global Catalog configuration
- Active Directory domain and forest validation
- Active Directory and DNS service validation
- Internal DNS resolution
- External DNS resolution
- Domain-authenticated network profile validation
- Remote Desktop with Network Level Authentication
- Remote Management
- Windows Firewall retained with ICMPv4 Echo Request enabled for troubleshooting
- DNS forwarders reviewed; none configured
- Root-hints-based external DNS resolution confirmed
- IPv6 retained
- VMware Tools verification
- Time zone verification
- Windows Update baseline check
- Active Directory DNS record inspection
- Reverse lookup zone and PTR record administration
- DNS-only forward and reverse resolution validation
- Temporary A/CNAME administration and cleanup recorded in the lab notes
- Organizational Unit structure and Domain Controller placement
- Departmental user attributes, OU placement, and manager relationships
- Global Security groups and corrected departmental memberships
- Fictional employee onboarding, transfer, and offboarding
- Workstation internal DNS configuration with DHCP addressing retained
- Domain membership, DC discovery, and secure-channel validation
- Enabled computer object in the Finance Workstations OU
- Standard domain-user authentication and departmental group-token validation

The Phase 4–5 baseline is documented through September 16, 2026. Centralized Group Policy is the next technical phase and has not yet started hands-on. Later work includes DHCP, file services, PowerShell automation, server operations, security, monitoring, and integrated troubleshooting.

See the [server module overview](README.md), [screenshot guide](screenshots/README.md), and [workstation baseline](../01-windows-workstation/system-baseline.md) for related documentation. Supporting files are being brought to the same checkpoint separately.
