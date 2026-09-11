# System Baseline — NS-W11-01

## Purpose

This document records the technical baseline configuration of `NS-W11-01`, the Windows 11 Pro Finance workstation used in the Northstar Solutions enterprise IT homelab.

The baseline documents the workstation's identity, virtual hardware, local user configuration, network configuration, storage, local policy, and endpoint security state before later integration with centralized Northstar Solutions infrastructure.

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

`NS-W11-01` is connected to the VMware virtual network through `Ethernet0` and receives its IPv4 configuration dynamically through DHCP.

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

The workstation remains DHCP-configured at this stage.

Its network configuration will be reassessed during the later Active Directory integration phase, when `NS-W11-01` is prepared to use the Northstar Solutions internal DNS infrastructure and join the domain.

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

## Baseline Status

**Windows 11 Workstation Baseline — Complete**

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

`NS-W11-01` will remain the Northstar Solutions Finance workstation and will be used in later infrastructure modules for Active Directory domain integration, internal DNS configuration, centralized Group Policy, networking, security hardening, and enterprise administration.