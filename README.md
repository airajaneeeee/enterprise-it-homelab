# Enterprise IT Infrastructure & Support Homelab

## Overview

This repository documents my hands-on enterprise IT homelab for practicing real-world IT support, Windows endpoint administration, Windows Server administration, Active Directory, DNS, identity and access management, networking, troubleshooting, automation, and security.

The environment simulates the IT infrastructure of a fictional organization called **Northstar Solutions**.

The lab is being built progressively through learning, implementation, administration, validation, and troubleshooting. Configuration decisions, selected evidence, technical notes, and lessons learned are documented at meaningful checkpoints.

## Current Environment

| System | Role | Status |
|---|---|---|
| `NS-DC01` | Windows Server 2025 Standard Evaluation Domain Controller / DNS / DHCP Server | Operational |
| `NS-W11-01` | Windows 11 Pro Finance Workstation | Domain Joined |

### Active Directory and Networking

- Forest / Domain: `ad.northstarsolutions.com`
- NetBIOS Domain: `NORTHSTAR`
- Domain Controller / Global Catalog: `NS-DC01.ad.northstarsolutions.com`
- Domain Controller IPv4: `192.168.252.10`
- Virtual Network: VMware VMnet8 NAT, `192.168.252.0/24`
- Default Gateway: `192.168.252.2`
- DHCP Server: `NS-DC01`, `192.168.252.10`; VMware DHCP disabled
- Workstation Addressing: Windows DHCP reservation, `192.168.252.120`
- Workstation DNS Server: `192.168.252.10`, supplied through DHCP Option 006

`NS-DC01` uses its local DNS service through the IPv4 loopback address `127.0.0.1`. The workstation queries that service through the server's network address, `192.168.252.10`.

The workstation computer account is located in `Northstar > Workstations > Finance`. Domain Controller discovery, the workstation-domain secure channel, and a Finance domain-user session were validated after integration.

VMware continues to provide NAT and the gateway at `192.168.252.2`. DHCP responsibility moved to Windows Server during Phase 7; `NS-DC01` is not the router.

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

### Phase 6 — Centralized Group Policy

Implemented and validated separate computer and Finance user policies:

- `Northstar - Workstation Baseline` appeared in computer-scope `gpresult` output on `NS-W11-01`; the recorded `InactivityTimeoutSecs` value was `900`.
- `Northstar - Finance User Policy` appeared in user-scope `gpresult` output for `NORTHSTAR\ava.chen`.
- Computer and user policy results were reviewed separately to validate the intended scope.

**SIM-GPO-001 — Missing Finance OU Link** was a controlled Group Policy troubleshooting scenario. The Finance policy stopped applying after its OU link was removed. The user location, existing GPO, and configured setting were checked before the missing link was identified. Restoring the link and refreshing policy restored the GPO in `gpresult`; the notes also record restoration of the functional restriction.

This was a simulated support scenario, not an unexpected production failure. See the retained [computer policy](02-windows-server/screenshots/15-workstation-baseline-gpo-verification.png), [Finance user policy](02-windows-server/screenshots/16-finance-user-gpo-verification.png), and [link-troubleshooting results](02-windows-server/screenshots/17-simulated-gpo-link-troubleshooting.png).

### Phase 7 — Windows DHCP

Installed the DHCP Server role on `NS-DC01`, completed post-installation configuration, and verified Active Directory authorization with `Get-DhcpServerInDC`.

The `Northstar Client Network` scope was configured as:

| Setting | Value |
|---|---|
| Scope ID / Subnet | `192.168.252.0/24` |
| Scope Range | `192.168.252.20–192.168.252.199` |
| Exclusion | `192.168.252.20–192.168.252.49` |
| Client Address Range after Exclusion | `192.168.252.50–192.168.252.199`, including the reservation below |
| Lease Duration | 8 days |
| Option 003 — Router | `192.168.252.2` |
| Option 006 — DNS Servers | `192.168.252.10` |
| Option 015 — DNS Domain Name | `ad.northstarsolutions.com` |
| Workstation Reservation | `NS-W11-01` → `192.168.252.120` |

The Windows scope remained inactive until VMware DHCP was disabled. VMware NAT stayed active. The client was set to obtain DNS automatically so that the DHCP-delivered DNS option could be tested rather than masked by its earlier manual setting.

The workstation initially received `192.168.252.50` from Windows DHCP and later received the reserved address `192.168.252.120` while remaining DHCP-enabled. DHCP Manager and PowerShell were used to inspect authorization, scope state, options, leases, and the reservation.

Two distinct troubleshooting activities were recorded:

- **Unexpected time / secure-channel issue:** `Test-ComputerSecureChannel` returned `False` despite working DNS and DC discovery. Incorrect workstation time was investigated as a contributing condition. After time correction and synchronization, the retained validation shows a closely synchronized clock and a successful secure-channel check. This was an unexpected lab issue; the evidence does not establish that DHCP caused it.
- **SIM-DHCP-001 — Incorrect DNS Option:** Option 006 was deliberately changed to `1.1.1.1`. The client retained a valid lease and public DNS access, but internal AD names failed. A direct query to `192.168.252.10` succeeded, isolating the client DNS configuration. Restoring Option 006, renewing the lease, and clearing the resolver cache restored internal resolution; the recovery screenshot also shows successful DC discovery and a healthy secure channel.

The first simulated-fault attempt used VMware DNS at `192.168.252.2`, but internal queries succeeded during that test. It was recorded as a successful test result, not falsely described as a failure. This later observation does not replace the earlier Phase 5 DNS findings.

## Current Architecture

```text
Northstar Solutions — ad.northstarsolutions.com
│
├── NS-DC01 — Windows Server 2025
│   ├── AD DS / DNS / DHCP / Global Catalog
│   └── 192.168.252.10
│
└── NS-W11-01 — Windows 11 Pro Finance workstation
    ├── Domain joined; Windows DHCP reservation → 192.168.252.120
    ├── DHCP / DNS → 192.168.252.10
    ├── Gateway → 192.168.252.2 (VMware NAT)
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

### Phase 6–7 Troubleshooting Reports

| Report | Classification | Outcome |
|---|---|---|
| [SIM-GPO-001 — Missing Finance User OU Link](02-windows-server/troubleshooting/SIM-GPO-001-missing-finance-ou-link.md) | Controlled simulated support incident | Restored the OU link and verified Finance policy application |
| [SIM-DHCP-001 — Incorrect DHCP DNS Option](02-windows-server/troubleshooting/SIM-DHCP-001-incorrect-dns-option.md) | Controlled simulated support incident | Restored internal DNS configuration and verified client recovery |
| [INC-004 — Workstation Time and Secure-Channel Validation Issue](02-windows-server/troubleshooting/INC-004-workstation-time-secure-channel.md) | Genuine unexpected lab incident | Secure-channel checks succeeded after clock correction and resynchronization; the initiating cause remained unconfirmed |

The reports include investigation, remediation, verification, and links to retained evidence. Earlier workstation cases are listed in the [workstation module](01-windows-workstation/README.md#troubleshooting-cases).

## Planned Next Phases

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

The current environment hosts AD DS, DNS, and DHCP on one Windows Server VM on a single physical host. Production redundancy, DHCP failover, enterprise scale, and future technologies are not claimed as implemented capabilities.

## Current Project Status

**Technical checkpoint: Phases 1–7 complete. Phase 8 — File Services and Permissions is next.**

This overview and the linked module overviews, system baselines, screenshot guides, knowledge bases, and interview preparation reflect the Phase 6–7 checkpoint. Earlier assessments and phase summaries preserve their historical configuration. The three Phase 6–7 troubleshooting reports are documented separately above.
