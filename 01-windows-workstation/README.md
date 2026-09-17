# Windows 11 Enterprise Workstation Administration

## Business Scenario

Northstar Solutions prepared a Windows 11 Pro workstation for a Finance department employee and later integrated it into the organization's Active Directory domain.

As the assigned junior IT support technician, I am responsible for preparing the workstation, managing local and domain-user access, validating system health and connectivity, reviewing endpoint security controls, troubleshooting common issues, and documenting the completed work.

## Workstation

- Computer Name: `NS-W11-01`
- Operating System: Windows 11 Pro
- Platform: VMware
- Department: Finance
- Role: End-user workstation
- Domain: `ad.northstarsolutions.com`
- Computer OU: `Northstar > Workstations > Finance`
- Addressing: VMware DHCP; IPv4 DNS server `192.168.252.10`

## Lab Objectives

- Establish a standardized workstation identity
- Configure administrator and standard-user accounts
- Inspect hardware and device drivers
- Review system performance
- Inspect Windows services
- Validate network configuration
- Troubleshoot common DNS and DHCP issues
- Review Windows system-integrity assessment and repair tools
- Review storage configuration
- Review Windows Defender and Firewall
- Assess BitLocker status
- Understand User Account Control
- Document troubleshooting and validation evidence
- Integrate the workstation with Northstar Active Directory
- Validate domain membership, Domain Controller discovery, and workstation trust
- Verify a Finance domain-user session and departmental group membership

## Completed Work

### Workstation Configuration and Access Control

- Standardized the Windows 11 workstation as `NS-W11-01` using the Northstar Solutions device naming convention.
- Created dedicated local administrator (`labadmin`) and standard Finance user (`finance.user`) accounts.
- Configured local group membership and applied the principle of least privilege.
- Tested User Account Control (UAC) and administrative elevation using authorized administrator credentials.
- Verified workstation identity, user context, and administrator membership using Windows administrative tools and command-line utilities.

### Device and Driver Administration

- Inspected physical and virtual devices using Windows Device Manager.
- Reviewed device status and driver information, including provider, version, date, and digital signer.
- Discovered an unidentified `Base System Device` reporting Device Manager Code 28.
- Investigated PCI Hardware IDs to identify the affected VMware virtual device.
- Determined that VMware Tools was missing and installed the required VMware guest components.
- Restarted the workstation and verified that the unidentified device and Code 28 warning were resolved.

### Performance and Process Troubleshooting

- Established a workstation performance baseline using Task Manager.
- Analyzed CPU, memory, disk, and network utilization through the Processes and Performance views.
- Investigated sustained high CPU utilization by sorting processes and examining Microsoft Edge child processes and PIDs.
- Tested a low-impact remediation by closing high-resource browser tabs before escalating to closing the application.
- Verified CPU utilization decreased from approximately 47–50% to approximately 11% after removing the active browser workload.
- Reviewed startup applications and considered application purpose and business requirements before making configuration changes.

### Windows Services Administration

- Inspected and administered Windows services using Services (`services.msc`).
- Reviewed service status, startup type, executable path, and dependencies.
- Practiced stopping and restarting the Print Spooler service.
- Verified service state using both Command Prompt and PowerShell.
- Used `sc query` and `Get-Service` to validate service status before and after configuration changes.

### Local Group Policy

- Reviewed Local Group Policy using `gpedit.msc`.
- Configured a workstation security policy preventing Remote Desktop Connection passwords from being saved.
- Applied policy changes using `gpupdate /force`.
- Reviewed Group Policy diagnostic tools including `gpresult` and Resultant Set of Policy (`rsop.msc`).
- Practiced interpreting policy states such as Not Configured, Enabled, and Disabled.

### Network Configuration and Troubleshooting

- Inspected detailed TCP/IP configuration using `ipconfig /all`.
- Identified the workstation IPv4 address, subnet mask, default gateway, DHCP server, and DNS server.
- Practiced DNS troubleshooting using `nslookup` and Windows DNS cache commands.
- Reviewed DHCP lease troubleshooting using `ipconfig /release` and `ipconfig /renew`.
- Distinguished DNS name-resolution problems from general network-connectivity problems.

### Windows System Integrity and Maintenance

- Reviewed Windows image health and system-file integrity using DISM and System File Checker.
- Practiced Windows image assessment using `DISM /Online /Cleanup-Image`.
- Reviewed the use of `sfc /scannow` for validating protected Windows system files.
- Reviewed Windows storage-management and cleanup tools.
- Examined Windows Update configuration, update history, advanced options, and the distinction between quality and feature updates.

### Storage Administration

- Provisioned additional virtual storage for the Finance workstation.
- Initialized and configured a secondary disk using Windows Disk Management.
- Created and formatted an NTFS data volume.
- Assigned drive letter `F:` and volume label `Finance-Data`.
- Verified the volume reported a Healthy status and confirmed read/write access through File Explorer.
- Reviewed GPT and MBR partitioning concepts and NTFS, exFAT, and ReFS file-system use cases.

### Endpoint Security Assessment

- Reviewed Microsoft Defender Antivirus and Windows Security configuration.
- Used PowerShell to inspect Microsoft Defender protection status.
- Reviewed Windows Defender Firewall profiles, inbound and outbound rules, and advanced firewall management.
- Used PowerShell to inspect firewall and network-profile configuration.
- Assessed BitLocker encryption status using `manage-bde -status`.
- Identified that BitLocker protection is currently disabled and recorded encryption at rest as a future endpoint-hardening consideration.
- Reviewed User Account Control and least-privilege administrative practices.

### Active Directory Domain Integration

During Phase 5, `NS-W11-01` was integrated into the domain hosted by `NS-DC01`.

- Recorded the pre-join `WORKGROUP` state and network configuration.
- Observed successful IP connectivity to the DC while Northstar domain and LDAP SRV lookups failed through VMware DNS.
- Changed the workstation's IPv4 DNS server from `192.168.252.2` to `192.168.252.10`, retaining VMware DHCP addressing.
- Validated internal DNS, AD service discovery, and external resolution according to the lab notes.
- Joined `ad.northstarsolutions.com`.
- Verified domain membership, Domain Controller discovery, and a healthy secure channel in an administrative session.
- Verified the enabled computer object in the Finance Workstations OU.
- Validated a separate session as `NORTHSTAR\ava.chen`, including `GG-Finance-Users` in the user token.
- Reviewed the displayed token and local Administrators membership to support the standard-user session.

The domain Finance identity is separate from the original local `finance.user` account. The DNS prerequisite observation is not presented as a deliberately induced failure.

The [workstation baseline](system-baseline.md) preserves the standalone configuration and adds the later integration checkpoint. The retained [domain membership](../02-windows-server/screenshots/12-ns-w11-01-domain-membership-verification.png), [computer-object placement](../02-windows-server/screenshots/13-ns-w11-01-ad-computer-object-verification.png), and [Finance session](../02-windows-server/screenshots/14-domain-user-login-and-group-membership.png) screenshots are maintained with the server module.

## Troubleshooting Cases

### INC-001 — Administrative Access and Least Privilege

Investigated a standard Finance user's inability to perform an administrative operation and verified that the restriction resulted from intentional standard-user permissions. Used authorized administrator elevation rather than granting permanent administrative access.

See [INC-001](troubleshooting/INC-001-user-permissions.md). The case documents an intentional access restriction; its unexpected-versus-simulated origin is not established by the record.

### INC-002 — Missing VMware Device Driver

Investigated an unidentified device reporting Code 28, used Hardware IDs to identify the VMware virtual hardware, determined that VMware Tools was missing, installed the required guest components, and verified successful device recognition after restart.

See [INC-002](troubleshooting/INC-002-missing-vmware-device-driver.md), an unexpected finding during baseline inspection.

### INC-003 — High CPU Utilization

Investigated workstation performance degradation using Task Manager, isolated significant CPU utilization to active Microsoft Edge workloads, tested corrective actions, and verified a substantial reduction in CPU utilization after remediation.

See [INC-003](troubleshooting/INC-003-high-cpu-edge-workload.md). The case uses a simulated Finance user report with observed performance measurements; it is not presented as an unexpected production incident.

## Tools and Technologies Used

- Windows 11 Pro
- VMware Workstation
- VMware Tools
- Windows Settings
- Computer Management
- Local Users and Groups
- User Account Control (UAC)
- Device Manager
- Task Manager
- Windows Services
- Local Group Policy Editor
- Resultant Set of Policy
- Command Prompt
- PowerShell
- Disk Management
- Microsoft Defender Antivirus
- Windows Defender Firewall
- Windows Security
- DISM
- System File Checker
- Active Directory domain integration
- `nltest`
- `Resolve-DnsName`
- `Test-ComputerSecureChannel`

## Commands Used

### Command Prompt

```cmd
hostname
whoami
net user
net localgroup administrators
devmgmt.msc
taskmgr
services.msc
sc query spooler
gpedit.msc
gpupdate /force
gpresult /r
rsop.msc
ipconfig /all
ipconfig /displaydns
ipconfig /flushdns
ipconfig /release
ipconfig /renew
nslookup
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
sfc /scannow
diskmgmt.msc
manage-bde -status
```

### PowerShell

```powershell
Get-Service Spooler
Get-MpComputerStatus
Get-NetFirewallProfile
Get-NetConnectionProfile
```

### Domain Integration Validation

Administrative validation on the workstation used:

```powershell
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
nltest /dsgetdc:ad.northstarsolutions.com
Test-ComputerSecureChannel -Verbose
```

The Finance domain-user session was checked separately:

```powershell
whoami
whoami /user
whoami /groups
$env:USERDOMAIN
$env:USERDNSDOMAIN
net localgroup Administrators
```

## Documentation

Supporting technical documentation for this workstation includes:

- [System configuration and workstation baseline](system-baseline.md)
- [Device and driver assessment](device-baseline.md)
- [Workstation screenshot evidence](screenshots/README.md)
- [Windows 11 administration knowledge base](../knowledge-base/windows-11-administration.md)
- [Workstation interview preparation](../interview-prep/windows-workstation.md)
- [Server module and domain integration](../02-windows-server/README.md)
- [Server screenshot guide](../02-windows-server/screenshots/README.md)

Historical endpoint assessments remain associated with the standalone baseline. The later domain-integration checks do not constitute a fresh hardware, storage, or endpoint-security assessment.

## Module Status

**Windows 11 Workstation Administration — Standalone Baseline and Domain Integration Complete**

Completed areas:

- Workstation identity and baseline configuration
- Local account and administrator management
- Least-privilege access control and UAC
- Device and driver administration
- Device Manager troubleshooting and VMware Tools remediation
- Task Manager and performance troubleshooting
- Windows Services administration
- Local Group Policy configuration
- DNS and DHCP troubleshooting fundamentals
- Windows system-integrity assessment
- Storage provisioning and NTFS volume administration
- Windows maintenance and update concepts
- Microsoft Defender assessment
- Windows Firewall administration
- BitLocker status assessment
- Technical documentation and troubleshooting case development
- Internal Active Directory DNS configuration
- Domain membership and Domain Controller discovery
- Workstation-domain secure-channel validation
- Enabled computer object in the Finance Workstations OU
- Finance domain-user authentication and departmental group-token validation

The workstation is now domain joined. Phase 6 — Centralized Group Policy is next and has not yet started hands-on. The earlier Local Group Policy exercise remains distinct from that future centralized policy work.

This overview reflects the recorded checkpoint through September 16, 2026. The workstation will continue to support later networking, security hardening, and enterprise administration modules.
