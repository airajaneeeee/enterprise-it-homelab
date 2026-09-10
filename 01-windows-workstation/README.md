# Windows 11 Enterprise Workstation Administration

## Business Scenario

Northstar Solutions is preparing a Windows 11 Pro workstation for a Finance department employee.

As the assigned junior IT support technician, I am responsible for preparing the workstation, standardizing its configuration, managing local user access, validating system health and network connectivity, reviewing endpoint security controls, troubleshooting common issues, and documenting the completed work.

## Workstation

- Computer Name: NS-W11-01
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
- Verify Windows system integrity
- Review storage configuration
- Review Windows Defender and Firewall
- Assess BitLocker status
- Understand User Account Control
- Document troubleshooting and validation evidence

## Status

In Progress

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
- Windows Settings
- Computer Management
- Local Users and Groups
- User Account Control (UAC)
- Device Manager
- Task Manager
- VMware Tools
- Command Prompt

## Commands Used

```cmd
hostname
whoami
net user
net localgroup administrators
devmgmt.msc
taskmgr
```

## Current Status

**In Progress**

Completed so far:

- Workstation identity and baseline configuration
- Local user and administrator management
- Least-privilege configuration
- UAC and administrative elevation
- Device and driver assessment
- Unknown-device and Code 28 troubleshooting
- VMware Tools installation and verification
- Task Manager resource analysis
- Process and PID investigation
- Startup application review
- Performance troubleshooting

Next:

- Windows Services and system utilities
- Local Group Policy
- Network troubleshooting
- Windows system repair
- Storage administration
- Windows Update
- Endpoint security and hardening