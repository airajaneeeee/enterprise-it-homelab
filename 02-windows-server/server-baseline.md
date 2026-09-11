# System Baseline — NS-DC01

## Purpose

This document records the technical baseline and infrastructure configuration of `NS-DC01`, the Windows Server 2025 system used in the Northstar Solutions enterprise IT homelab.

The baseline documents the server's identity, virtual hardware, network configuration, pre-deployment state, Active Directory Domain Services configuration, DNS configuration, and post-deployment validation.

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

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.10` |
| Subnet Mask | `255.255.255.0` |
| Prefix Length | `/24` |
| Default Gateway | `192.168.252.2` |
| Preferred DNS | `192.168.252.10` |
| Alternate DNS | None |
| DHCP Enabled | No |

The server uses its own IPv4 address as its preferred DNS server because it now hosts DNS for the Northstar Solutions Active Directory domain.

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

### Before and After Comparison

| Test | Before DNS Deployment | After DNS Deployment |
|---|---|---|
| Gateway connectivity | Successful | Available |
| External IP connectivity | Successful | Available |
| Query DNS server at `192.168.252.10` | No response | Operational |
| Internal AD DNS resolution | Not available | Successful |
| External DNS resolution | Failed through `192.168.252.10` | Successful |

The comparison demonstrates the change from a server configured to query a DNS service that had not yet been deployed to an operational Active Directory DNS server capable of resolving both the internal domain namespace and external DNS names.

## Infrastructure Role

`NS-DC01` currently provides:

- Active Directory Domain Services
- Domain authentication infrastructure
- DNS services
- Global Catalog services

Current architecture:

```text
Northstar Solutions
└── ad.northstarsolutions.com
    └── NS-DC01
        ├── Windows Server 2025
        ├── Domain Controller
        ├── Active Directory Domain Services
        ├── DNS Server
        ├── Global Catalog
        └── 192.168.252.10
```

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

**Windows Server and Active Directory Baseline — Complete**

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

`NS-DC01` will continue to serve as the primary infrastructure server for later Northstar Solutions modules involving Organizational Units, domain users and groups, domain-joined workstations, centralized Group Policy, DHCP, file services, PowerShell administration, security, and infrastructure troubleshooting.