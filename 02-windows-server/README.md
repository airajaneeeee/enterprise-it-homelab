# Windows Server Infrastructure Administration

## Status

Windows Server Baseline — Complete  
Active Directory and DNS Deployment — Next

## Business Scenario

Northstar Solutions is expanding from standalone workstation administration to centralized Windows infrastructure.

A Windows Server 2025 system was deployed in VMware Workstation to provide the foundation for future enterprise services including Active Directory Domain Services, DNS, Group Policy, centralized identity management, and domain-based workstation administration.

The server was configured and validated at the operating-system and network level before introducing domain services.

## Server Overview

| Item | Configuration |
|---|---|
| Server Name | `NS-DC01` |
| Operating System | Windows Server 2025 Standard Evaluation |
| Platform | VMware Workstation |
| vCPU | 2 |
| Memory | 4 GB |
| Virtual Disk | ~60 GB |
| Network | VMware VMnet8 / NAT |
| IPv4 Address | `192.168.252.10` |
| Planned Role | Domain Controller / DNS Server |

## Objectives

- Deploy Windows Server 2025 in a virtualized lab environment.
- Establish a standardized server naming convention.
- Examine the underlying VMware virtual network before assigning server addressing.
- Configure predictable static IPv4 addressing for core infrastructure.
- Validate local gateway and external IP connectivity.
- Prepare DNS client configuration for the future Active Directory environment.
- Establish a documented pre-domain baseline before installing infrastructure roles.

## Completed Work

### 1. Windows Server Deployment

Created a dedicated Windows Server 2025 virtual machine in VMware Workstation.

The server was configured with resources appropriate for the available homelab environment and renamed:

`NS-DC01`

Naming convention:

- `NS` — Northstar Solutions
- `DC` — planned Domain Controller role
- `01` — first server assigned to this role

At this stage, the name identifies the server's intended role. The system does not become an actual Domain Controller until Active Directory Domain Services is installed and the server is promoted.

### 2. Network Baseline Assessment

Before assigning a static address, the VMware VMnet8 network was inspected.

Identified configuration:

| Network Setting | Value |
|---|---|
| Network | `192.168.252.0/24` |
| Subnet Mask | `255.255.255.0` |
| VMware NAT Gateway | `192.168.252.2` |
| VMware DHCP Range | `192.168.252.128 - 192.168.252.254` |
| Initial Server Address | `192.168.252.129` |

The original server address was dynamically assigned from the VMware DHCP pool.

### 3. Static IPv4 Configuration

The server was changed from DHCP to a manually configured IPv4 address.

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.10` |
| Subnet Mask | `255.255.255.0` |
| Prefix Length | `/24` |
| Default Gateway | `192.168.252.2` |
| Preferred DNS | `192.168.252.10` |
| DHCP | Disabled |

`192.168.252.10` was selected outside the VMware DHCP allocation range to provide predictable addressing for the infrastructure server and reduce the risk of a DHCP allocation conflict.

### 4. Connectivity Validation

After static addressing was configured, connectivity was tested.

The VMware gateway responded successfully:

`ping 192.168.252.2`

External IP connectivity was also verified:

`ping 8.8.8.8`

The external test returned replies with no packet loss, confirming that IP routing remained functional after the static configuration change.

### 5. Pre-DNS Validation

The server's preferred DNS address was configured as:

`192.168.252.10`

This points `NS-DC01` to itself in preparation for hosting the Windows DNS Server role.

A DNS query was tested before DNS Server deployment:

`nslookup microsoft.com`

The query returned no response from `192.168.252.10`.

This result was expected because the server had been configured to query itself for DNS, but the DNS Server role had not yet been installed.

This provides a pre-deployment baseline that can later be compared with DNS functionality after the DNS service is configured.

## Infrastructure Design Decisions

### Static Addressing

Core infrastructure services require predictable network addressing so dependent systems can reliably locate them.

The homelab therefore uses a static address for `NS-DC01` rather than relying on a dynamically assigned VMware DHCP lease.

### DNS Design

The planned Active Directory Domain Controller will also provide DNS services for the Northstar domain environment.

Domain-connected Windows systems will use the internal DNS service so they can locate Active Directory services and internal resources.

External DNS resolution will be validated after the DNS Server role is deployed and configured.

## Production Considerations

This environment applies production-oriented infrastructure concepts while adapting them to the limitations of a single-host VMware homelab.

A production environment would additionally consider:

- Formal IP address management and network documentation
- Dedicated server VLANs or network segments
- DHCP exclusions or reservations coordinated with network infrastructure
- Multiple Domain Controllers and DNS servers for redundancy
- Monitoring and alerting
- Backup and disaster-recovery procedures
- Change-management processes
- Security baselines and hardening standards
- Controlled administrative access

The homelab will implement applicable concepts while documenting where its architecture differs from a production environment.

## Next Phase

The next infrastructure phase will deploy:

- Active Directory Domain Services (AD DS)
- DNS Server
- A new Active Directory forest and domain
- Domain Controller services on `NS-DC01`

After deployment, AD DS and DNS functionality will be validated before users, groups, Organizational Units, Group Policy, or domain-joined workstations are introduced.