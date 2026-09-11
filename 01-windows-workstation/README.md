# Windows 11 Enterprise Workstation Administration

## Business Scenario

Northstar Solutions is preparing a Windows 11 Pro workstation for a Finance department employee.

As the assigned junior IT support technician, I am responsible for preparing the workstation, standardizing its configuration, managing local user access, validating system health and network connectivity, reviewing endpoint security controls, troubleshooting common issues, and documenting the completed work.

## Workstation

- Computer Name: `NS-W11-01`
- Operating System: Windows 11 Pro
- Platform: VMware
- Department: Finance
- Role: End-user workstation

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

## Troubleshooting Cases

### INC-001 — Administrative Access and Least Privilege

Investigated a standard Finance user's inability to perform an administrative operation and verified that the restriction resulted from intentional standard-user permissions. Used authorized administrator elevation rather than granting permanent administrative access.

### INC-002 — Missing VMware Device Driver

Investigated an unidentified device reporting Code 28, used Hardware IDs to identify the VMware virtual hardware, determined that VMware Tools was missing, installed the required guest components, and verified successful device recognition after restart.

### INC-003 — High CPU Utilization

Investigated workstation performance degradation using Task Manager, isolated significant CPU utilization to active Microsoft Edge workloads, tested corrective actions, and verified a substantial reduction in CPU utilization after remediation.

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

## Documentation

Supporting technical documentation for this workstation includes:

- System configuration and workstation baseline
- Device and driver assessment
- Configuration and validation screenshots
- Documented troubleshooting incidents
- Windows 11 administration knowledge base
- Interview preparation based on completed lab work

## Module Status

**Windows 11 Workstation Administration — Complete**

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

The workstation will continue to be used in later Northstar Solutions infrastructure modules for domain integration, centralized policy management, networking, security hardening, and enterprise administration.