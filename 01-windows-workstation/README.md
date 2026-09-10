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

### Workstation Identity and Local Account Administration

- Standardized the Windows 11 workstation hostname as `NS-W11-01`.
- Created a dedicated local administrator account (`labadmin`) for IT administrative tasks.
- Created a standard Finance employee account (`finance.user`) for day-to-day user activity.
- Configured local group membership to separate administrative and standard-user privileges.
- Applied the principle of least privilege by keeping the Finance employee account out of the local Administrators group.
- Tested User Account Control (UAC) and administrative elevation using authorized administrator credentials.
- Verified user identity and local group membership using Windows administrative tools and command-line utilities.
- Investigated an access-denied scenario caused by insufficient privileges and validated the appropriate elevation process.

### Tools and Commands Used

- Computer Management
- Local Users and Groups
- Command Prompt
- User Account Control (UAC)
- `hostname`
- `whoami`
- `net user`
- `net localgroup administrators`