# System Baseline — NS-W11-01

## Purpose

This document records the technical baseline configuration of `NS-W11-01`, the Windows 11 Pro Finance workstation used in the Northstar Solutions enterprise IT homelab.

The baseline preserves the workstation's original standalone identity, hardware, local accounts, networking, storage, policy, and security observations. A separate domain-integration checkpoint records the later transition into the Northstar Active Directory environment through September 16, 2026.

Original configuration tables describe the standalone baseline unless identified as part of the later checkpoint. The integration notes do not establish a fresh hardware, storage, update, or endpoint-security assessment.

## System Identity

| Item | Value |
|---|---|
| VMware VM Name | `Northstar-W11-Client01` |
| Windows Hostname | `NS-W11-01` |
| Operating System | Windows 11 Pro |
| Windows Version | 25H2 |
| OS Build | 26200.9445 |
| Platform | VMware Workstation |
| Assigned Role | Finance Department Workstation |

## Virtual Hardware

| Component | Value |
|---|---|
| Processor / vCPU | 2 vCPUs — 12th Gen Intel(R) Core(TM) i7-12700H (2.69 GHz) |
| Installed RAM | 8.00 GB |
| System Type | 64-bit operating system, x64-based processor |

These values are retained from the original workstation assessment. Later planning notes refer to approximately 4 GB of workstation RAM, but a subsequent allocation change has not been confirmed. The historical 8 GB observation is therefore preserved; current allocation should be verified before making capacity-dependent changes.

## Workstation Identity Configuration

### Original State

The workstation initially used the Windows computer name created during the original setup.

### Configuration Change

The workstation was renamed to:

`NS-W11-01`

### Naming Convention

- `NS` = Northstar Solutions
- `W11` = Windows 11 workstation
- `01` = Workstation 01

### Verification

The workstation hostname was verified using:

```cmd
hostname
```

Verified result:

```text
NS-W11-01
```

The Windows **Settings > System > About** page was also reviewed to confirm the workstation hostname, operating system edition, Windows version, OS build, installed memory, processor information, and system architecture.

## Local User Configuration

Two dedicated local accounts were configured for the workstation.

| Account | Purpose | Access Level |
|---|---|---|
| `labadmin` | IT administration | Local Administrator |
| `finance.user` | Simulated Finance employee | Standard User |

### Administrative Account

`labadmin` was added to the local `Administrators` group and is used for privileged IT administration tasks.

Administrative context and local administrator membership were inspected using:

```cmd
whoami
net localgroup administrators
```

### Standard Employee Account

`finance.user` was configured as a standard user and was not added to the local `Administrators` group.

This configuration separates normal employee activity from privileged administrative operations and supports the principle of least privilege.

Administrative operations performed from the standard-user context require authorized elevation rather than permanent administrator membership.

## Network Baseline

At the standalone baseline, `NS-W11-01` was connected to the VMware virtual network through `Ethernet0` and received its IPv4 configuration dynamically through DHCP. The table preserves that earlier state, including VMware DNS.

| Setting | Value |
|---|---|
| Network Adapter | `Ethernet0` |
| Adapter Description | Intel(R) 82574L Gigabit Network Connection |
| IPv4 Address | `192.168.252.128` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.254` |
| DNS Server | `192.168.252.2` |
| DNS Suffix | `localdomain` |
| DHCP Enabled | Yes |

Detailed network configuration was inspected using:

```cmd
ipconfig /all
```

DNS and DHCP administration and troubleshooting commands practiced during the workstation module included:

```cmd
nslookup
ipconfig /displaydns
ipconfig /flushdns
ipconfig /release
ipconfig /renew
```

The workstation was DHCP-configured at this stage. The recorded IPv4 address is a lease observation, not a permanent address assignment.

During the later domain-integration checkpoint below, DHCP addressing was retained and the IPv4 DNS server was changed to `192.168.252.10`. The historical `192.168.252.2` DNS value and `localdomain` suffix above are not presented as the post-join configuration.

## Local Policy Configuration

Local Group Policy was reviewed and configured using:

```cmd
gpedit.msc
```

The following workstation policy was configured:

**Do not allow passwords to be saved**

for the Remote Desktop Connection client.

The policy change was applied using:

```cmd
gpupdate /force
```

Group Policy diagnostic tools reviewed during the workstation module included:

```cmd
gpresult /r
rsop.msc
```

This local policy exercise introduced standalone Windows policy administration before centralized domain-based Group Policy is implemented later in the Northstar Solutions environment.

## Storage Configuration

| Volume | Purpose | File System | Capacity | Status |
|---|---|---|---|---|
| `C:` | Windows operating system | NTFS | 952.60 GB | Healthy |
| `Finance-Data (F:)` | Simulated Finance department data | NTFS | ~10 GB | Healthy |

A secondary virtual disk was provisioned and configured using Windows Disk Management.

The disk was initialized, formatted using NTFS, assigned drive letter `F:`, and labeled:

`Finance-Data`

Read/write access to the new volume was verified through Windows File Explorer.

## Endpoint Security Baseline

| Control | Assessment |
|---|---|
| Microsoft Defender Antivirus | Verified Active |
| Real-Time Protection | Verified Enabled |
| Windows Defender Firewall | Verified Enabled |
| User Account Control | Tested |
| Least-Privilege User Configuration | Implemented |
| BitLocker | Protection Off |

Microsoft Defender and Windows Firewall configuration were reviewed using Windows Security, Windows administrative tools, and PowerShell.

Relevant PowerShell commands included:

```powershell
Get-MpComputerStatus
Get-NetFirewallProfile
Get-NetConnectionProfile
```

### BitLocker Assessment

BitLocker status was assessed using:

```cmd
manage-bde -status
```

The operating-system volume reported:

| Item | Result |
|---|---|
| Conversion Status | Fully Decrypted |
| Percentage Encrypted | 0.0% |
| Protection Status | Protection Off |
| Lock Status | Unlocked |
| Key Protectors | None Found |

BitLocker protection is therefore not currently enabled on `NS-W11-01`.

Encryption at rest has been identified as a future endpoint-hardening consideration pending evaluation of virtual TPM and recovery-key configuration.

## System Integrity and Maintenance

Windows image-health and protected system-file assessment tools were reviewed and practiced using:

```cmd
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
sfc /scannow
```

These commands were included as part of workstation maintenance and system-integrity practice.

No specific final DISM or SFC health result is recorded in this baseline.

Windows Update configuration, update history, advanced update options, storage-management tools, and system cleanup functionality were also reviewed during the workstation administration module.

## Domain Integration Checkpoint — September 16, 2026

This section records Phase 5 work from the supplied lab notes and retained validation screenshots. It supplements the standalone baseline rather than replacing its historical observations.

### Pre-Join Validation and DNS Configuration

Before joining the domain, the workstation was recorded as `WORKGROUP` with `CsPartOfDomain = False`.

| Setting | Pre-Join Observation | Integration State |
|---|---|---|
| IPv4 Assignment | VMware DHCP | VMware DHCP retained |
| Observed IPv4 Address | `192.168.252.128` | DHCP; no static address assigned |
| Subnet Mask | `255.255.255.0` | No change reported |
| Default Gateway | `192.168.252.2` | No change reported |
| DHCP Server | `192.168.252.254` | VMware DHCP retained |
| IPv4 DNS Server | `192.168.252.2` | `192.168.252.10` |

The notes record successful connectivity to `NS-DC01` at `192.168.252.10`, while the Northstar domain and LDAP SRV records did not resolve through VMware DNS. This demonstrated a DNS prerequisite gap despite working IP connectivity.

The workstation's IPv4 DNS server was changed to the DC's network address, `192.168.252.10`. The DC's local loopback setting `127.0.0.1` was not assigned to the workstation.

After the change, the notes record successful resolution of:

- `NS-DC01.ad.northstarsolutions.com`
- `ad.northstarsolutions.com`
- `_ldap._tcp.dc._msdcs.ad.northstarsolutions.com`
- External names such as `microsoft.com`

The pre-join findings and DNS change are written lab observations without a separate retained screenshot. They are not classified as a deliberately induced domain-join incident.

### Domain Membership and Secure Channel

The workstation was joined to `ad.northstarsolutions.com`. Validation in an administrative PowerShell session used:

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

These results are visible in the [domain-membership validation screenshot](../02-windows-server/screenshots/12-ns-w11-01-domain-membership-verification.png).

### Active Directory Computer Object

The enabled workstation computer object was moved to `Northstar > Workstations > Finance`.

Its verified Distinguished Name is:

```text
CN=NS-W11-01,OU=Finance,OU=Workstations,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
```

The [server-side computer-object verification](../02-windows-server/screenshots/13-ns-w11-01-ad-computer-object-verification.png) shows both the enabled state and OU placement. This location prepares the Finance workstation for future centralized policy targeting.

### Finance Domain-User Session

A separate session was validated as `NORTHSTAR\ava.chen` using:

```powershell
whoami
whoami /user
whoami /groups
$env:USERDOMAIN
$env:USERDNSDOMAIN
net localgroup Administrators
nltest /dsgetdc:ad.northstarsolutions.com
```

The [Finance session screenshot](../02-windows-server/screenshots/14-domain-user-login-and-group-membership.png) shows:

- The `NORTHSTAR\ava.chen` identity and user SID.
- `NORTHSTAR\GG-Finance-Users` in the user's Windows security token.
- `USERDOMAIN=NORTHSTAR`.
- `USERDNSDOMAIN=AD.NORTHSTARSOLUTIONS.COM`.
- Successful Domain Controller discovery.
- No individual Ava Chen entry in local Administrators and no `BUILTIN\Administrators` entry in the displayed token.

These results support the documented standard-user session and departmental membership. The domain environment variables describe the user session; they are not a separate verification of the network adapter's DNS suffix settings.

The secure-channel command at the bottom of that screenshot has no displayed result. The successful secure-channel result is documented in the separate administrative screenshot above.

### Local and Domain Identity Context

The original `NS-W11-01\labadmin` and `NS-W11-01\finance.user` entries describe local accounts from the standalone baseline. `NORTHSTAR\ava.chen` is a separate Active Directory identity, not a renamed local Finance account.

The later local Administrators output still includes `labadmin`. The integration evidence does not constitute a full reassessment of every existing local account or resource permission.

### Policy and Security Scope

The earlier Remote Desktop password-saving policy was a Local Group Policy exercise. The domain join and OU placement prepare the workstation for Phase 6; new centralized Group Policy work has not yet started hands-on.

The Phase 5 evidence establishes domain membership, workstation trust, and a standard domain-user session. It does not establish a new Defender, firewall, BitLocker, storage, or system-integrity assessment. Those values remain the historical observations recorded above.

## Baseline Status

**Windows 11 Workstation Baseline — Complete through Domain Integration**

The documented standalone workstation baseline includes:

- Standardized workstation identity
- Dedicated administrator and standard-user accounts
- Least-privilege access configuration
- DHCP-based network configuration
- Local Group Policy configuration
- NTFS operating-system and Finance data volumes
- Endpoint security assessment
- BitLocker encryption assessment
- Windows system-integrity and maintenance tool practice

The later domain-integration checkpoint adds:

- Pre-join network and DNS observations
- Internal DNS configuration with VMware DHCP addressing retained
- Membership in `ad.northstarsolutions.com`
- Domain Controller discovery and secure-channel validation
- Enabled computer object in the Finance Workstations OU
- Finance domain-user authentication and departmental token membership
- Standard-user session checks

`NS-W11-01` is now the domain-joined Northstar Finance workstation. Phase 6 — Centralized Group Policy is next; later work includes networking, security hardening, and further enterprise administration.

Related documentation: [workstation module](README.md), [server baseline](../02-windows-server/server-baseline.md), and [server screenshot guide](../02-windows-server/screenshots/README.md).
