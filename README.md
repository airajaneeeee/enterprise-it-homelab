# Enterprise IT Infrastructure & Support Homelab

## Overview

This repository documents my hands-on enterprise IT homelab for practicing real-world IT support, Windows endpoint administration, Windows Server administration, Active Directory, DNS, identity and access management, networking, troubleshooting, automation, and security.

The environment simulates the IT infrastructure of a fictional organization called **Northstar Solutions**.

The lab is being built progressively through learning, implementation, administration, validation, and troubleshooting. Configuration decisions, selected evidence, technical notes, and lessons learned are documented at meaningful checkpoints.

## Current Environment

| System | Role | Status |
|---|---|---|
| `NS-DC01` | Windows Server 2025 Standard Evaluation Domain Controller / DNS Server | Operational |
| `NS-W11-01` | Windows 11 Pro Finance Workstation | Domain Joined |

### Active Directory and Networking

- Forest / Domain: `ad.northstarsolutions.com`
- NetBIOS Domain: `NORTHSTAR`
- Domain Controller / Global Catalog: `NS-DC01.ad.northstarsolutions.com`
- Domain Controller IPv4: `192.168.252.10`
- Virtual Network: VMware VMnet8 NAT, `192.168.252.0/24`
- Default Gateway: `192.168.252.2`
- Workstation Addressing: VMware DHCP
- Workstation DNS Server: `192.168.252.10`

`NS-DC01` uses its local DNS service through the IPv4 loopback address `127.0.0.1`. The workstation queries that service through the server's network address, `192.168.252.10`.

The workstation computer account is located in `Northstar > Workstations > Finance`. Domain Controller discovery, the workstation-domain secure channel, and a Finance domain-user session were validated after integration.

## Completed Project Phases

### Phase 1 — Windows 11 Enterprise Workstation Administration

Built and administered a Windows 11 Pro workstation for a simulated Finance department environment.

Hands-on work included:

- Workstation identity and standardized naming
- Local administrator and standard-user configuration
- Least-privilege access control and UAC
- Device and driver troubleshooting
- VMware Tools deployment and verification
- Performance and process troubleshooting
- Windows Services administration
- Local Group Policy configuration
- DNS and DHCP troubleshooting fundamentals
- Windows system-integrity assessment
- Disk and NTFS volume administration
- Windows Update and maintenance concepts
- Microsoft Defender and Firewall assessment
- BitLocker security assessment

Three documented troubleshooting cases were completed involving administrative permissions, a missing VMware device driver, and high CPU utilization.

### Phase 2 — Windows Server Foundation

Deployed `NS-DC01` in VMware Workstation, standardized its identity, assessed the virtual network, and configured predictable static IPv4 addressing before introducing infrastructure services.

### Phase 3 — Active Directory Domain Services and DNS Foundation

Installed AD DS and DNS, created the `ad.northstarsolutions.com` forest and domain, and promoted `NS-DC01` as the first Domain Controller and Global Catalog.

Validated the domain, forest, core services, and internal and external DNS resolution. A post-deployment administration baseline covered remote administration, network profile, IPv6, firewall configuration, VMware Tools, time zone, and Windows Update checks.

### Phase 4 — Active Directory Administration

Completed administration involving:

- Inspection of Active Directory DNS structures and SRV records
- Reverse lookup zone and PTR record administration
- Forward and reverse DNS validation
- Temporary A and CNAME record practice followed by cleanup
- Organizational Unit design for users, workstations, groups, service accounts, and disabled objects
- Six fictional employee identities with departmental attributes and manager relationships
- Three departmental Global Security groups and membership validation
- A fictional employee onboarding, transfer, and offboarding exercise

The groups establish the Accounts → Global Groups portion of AGDLP. Resource-specific Domain Local groups and permissions remain future work.

### Phase 5 — Domain-Joined Workstation

Configured `NS-W11-01` to use Northstar internal DNS while retaining VMware DHCP addressing, then joined it to `ad.northstarsolutions.com`.

Post-join validation included:

- Domain membership and Domain Controller discovery
- A successful workstation-domain secure-channel check
- Enabled Active Directory computer object in the Finance Workstations OU
- Authentication as `NORTHSTAR\ava.chen`
- `GG-Finance-Users` membership in the authenticated user's security token
- Review of the user's token and local Administrators membership to validate the standard-user session

## Current Architecture

```text
Northstar Solutions — ad.northstarsolutions.com
│
├── NS-DC01 — Windows Server 2025
│   ├── AD DS / DNS / Global Catalog
│   └── 192.168.252.10
│
└── NS-W11-01 — Windows 11 Pro Finance workstation
    ├── Domain joined; VMware DHCP IPv4
    ├── DNS → 192.168.252.10
    └── Computer OU → Northstar / Workstations / Finance
```

## Documentation

| Document | Purpose |
|---|---|
| [Windows Workstation Module](01-windows-workstation/README.md) | Endpoint administration and troubleshooting cases |
| [Workstation Baseline](01-windows-workstation/system-baseline.md) | Workstation configuration and validation |
| [Windows Server Module](02-windows-server/README.md) | Infrastructure deployment and administration |
| [Server Baseline](02-windows-server/server-baseline.md) | Server and Active Directory configuration |
| [Workstation Screenshot Evidence](01-windows-workstation/screenshots/README.md) | Selected endpoint evidence and interpretation |
| [Server Screenshot Evidence](02-windows-server/screenshots/README.md) | Selected infrastructure evidence and interpretation |
| [Windows Knowledge Base](knowledge-base/windows-11-administration.md) | Reusable endpoint concepts and procedures |
| [Server Knowledge Base](knowledge-base/windows-server-administration.md) | Reusable server, identity, and DNS concepts |
| [Workstation Interview Preparation](interview-prep/windows-workstation.md) | Questions and answers based on endpoint work |
| [Server Interview Preparation](interview-prep/windows-server-interview.md) | Questions and answers based on infrastructure work |

## Planned Next Phases

- Phase 6 — Centralized Group Policy
- Phase 7 — DHCP administration
- Phase 8 — File services and permissions
- Phase 9 — PowerShell administration and automation
- Phase 10 — Server operations, monitoring, logging, backup, and recovery
- Phase 11 — Enterprise networking and VLANs
- Phase 12 — Linux administration
- Phase 13 — Microsoft 365, Entra ID, and selected Azure administration where hands-on access is available
- Phase 14 — Security hardening and investigation
- Phase 15 — Integrated IT support and infrastructure troubleshooting scenarios

## Project Approach

Northstar is a simulated enterprise homelab. The portfolio records work performed in that environment and distinguishes it from production experience.

Healthy configurations are built and verified before deliberately introducing controlled faults. Unexpected project incidents, simulated support incidents, and focused exercises are labeled according to what actually occurred.

Evidence is selective: screenshots and written observations demonstrate meaningful configuration, administration, validation, and troubleshooting rather than every individual click.

The current environment uses one Domain Controller and DNS server on a single physical host. Production redundancy, enterprise scale, and future technologies are not claimed as implemented capabilities.

## Current Project Status

**Technical checkpoint: Phases 1–5 complete. Phase 6 — Centralized Group Policy is next and has not yet started hands-on.**

This overview reflects the project notes and retained evidence through September 16, 2026. Supporting Markdown documentation is being updated to the same Phase 4–5 checkpoint; some linked documents still describe the earlier state.
