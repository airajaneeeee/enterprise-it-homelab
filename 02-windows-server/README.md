# Windows Server Infrastructure Administration

## Status

Windows Server Baseline — Complete  
Active Directory and DNS Deployment — Next

## Business Scenario

Northstar Solutions is expanding its lab environment from standalone Windows workstation administration to centralized Windows infrastructure.

A Windows Server 2025 system was deployed in VMware Workstation to establish the foundation for Active Directory Domain Services, DNS, Group Policy, centralized identity management, and domain-based workstation administration.

The server was configured and validated at the operating-system and network level before introducing domain services.

## Environment

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

## Work Completed

### Windows Server Deployment

- Created a dedicated Windows Server 2025 virtual machine in VMware Workstation.
- Configured virtual CPU, memory, storage, and networking for the lab environment.
- Renamed the server to `NS-DC01` using the Northstar Solutions naming convention.
- Verified the new Windows hostname after restart.

### Network Assessment

- Inspected the VMware VMnet8 NAT network before assigning server addressing.
- Identified the `192.168.252.0/24` network.
- Identified the VMware NAT gateway at `192.168.252.2`.
- Identified the VMware DHCP allocation range of `192.168.252.128-192.168.252.254`.
- Confirmed that the server initially received `192.168.252.129` through DHCP.

### Static IPv4 Configuration

Configured `NS-DC01` with:

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.10` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| Preferred DNS | `192.168.252.10` |
| DHCP | Disabled |

The server was assigned a predictable address outside the VMware DHCP allocation range in preparation for providing core infrastructure services.

### Connectivity Validation

Validated the new network configuration by:

- Successfully reaching the VMware NAT gateway.
- Successfully reaching an external IP address using `ping 8.8.8.8`.
- Testing DNS resolution separately with `nslookup`.
- Establishing a pre-DNS baseline before the DNS Server role is deployed.

The DNS query did not receive a response because `NS-DC01` was configured to query itself while the DNS Server role had not yet been installed. This result will be compared with post-deployment DNS validation.

## Skills Demonstrated

- Windows Server 2025 deployment
- VMware virtual machine administration
- Windows Server configuration
- IPv4 addressing and subnet identification
- DHCP versus static addressing
- DNS client configuration
- Network connectivity validation
- Command-line network troubleshooting
- Infrastructure documentation
- Production-oriented infrastructure planning

## Production Considerations

This homelab applies production-oriented practices while adapting them to a single-host virtual environment.

A production implementation would additionally consider formal IP address management, dedicated server networks or VLANs, redundant Domain Controllers and DNS servers, monitoring, backup and disaster recovery, security baselines, controlled administrative access, and formal change management.

## Next Phase

`NS-DC01` is prepared for:

- Active Directory Domain Services
- DNS Server
- Domain Controller promotion
- Active Directory domain deployment

AD DS and DNS will be validated before domain users, groups, Organizational Units, Group Policy, or domain-joined workstations are introduced.