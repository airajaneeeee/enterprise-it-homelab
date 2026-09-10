# Windows Workstation Administration — Interview Prep

This document contains interview questions and sample answers based on the hands-on Windows 11 administration and troubleshooting work completed in the Northstar Solutions enterprise IT homelab.

---

## 1. Windows Workstation, Local Accounts, and Access Control

### 1. What is a hostname?

A hostname is the name used to identify a computer on a system or network.

In my Northstar Solutions homelab, I standardized the Windows 11 workstation hostname as:

```text
NS-W11-01
```

The naming convention represents:

- `NS` — Northstar Solutions
- `W11` — Windows 11
- `01` — Workstation 01

I verified the hostname using:

```cmd
hostname
```

### 2. Why are standardized computer names useful?

Standardized computer names make systems easier to identify, manage, troubleshoot, inventory, and support.

For example, a naming convention can help administrators quickly determine information such as the organization, device type, location, department, or device number.

In my homelab, I used `NS-W11-01` as the standardized name for the first Windows 11 workstation.

### 3. What is the difference between a VMware VM name and a Windows hostname?

The VMware VM name identifies the virtual machine within the virtualization platform.

The Windows hostname identifies the computer within the Windows operating system and network environment.

In my lab:

- VMware VM name: `Northstar-W11-Client01`
- Windows hostname: `NS-W11-01`

They can be different because they belong to different management layers.

### 4. What does the `whoami` command show?

`whoami` displays the security identity under which the command is running.

For example:

```cmd
whoami
```

May return:

```text
ns-w11-01\labadmin
```

This shows that `labadmin` is a local account on workstation `NS-W11-01`.

### 5. What is the difference between a local account and a domain account?

A local account is created and authenticated by an individual Windows computer.

For example:

```text
NS-W11-01\labadmin
```

Is a local account stored on `NS-W11-01`.

A domain account is centrally created and managed through a directory service such as Active Directory and can be used across authorized domain-joined systems.

My current Windows workstation lab uses local accounts. I will implement domain accounts later when I build the Active Directory environment.

### 6. What is the difference between an administrator and a standard user?

An administrator account can perform privileged system operations such as installing certain software, modifying protected system settings, and managing users.

A standard user has more limited permissions and is intended for normal day-to-day work.

In my lab, I configured:

- `labadmin` — Local administrator
- `finance.user` — Standard Finance employee

This separated administrative work from normal user activity.

### 7. What is the principle of least privilege?

The principle of least privilege means users and processes should receive only the permissions required to perform their responsibilities.

For example, my simulated Finance employee account was configured as a standard user rather than a local administrator.

Administrative credentials can be used when an approved task requires elevation instead of permanently giving the employee administrator privileges.

### 8. Why should employees generally use standard-user accounts?

Giving users unnecessary administrator privileges increases security and operational risk.

An administrator account can make system-wide changes, install software, modify protected settings, and potentially affect security controls.

Using standard-user accounts reduces the impact of mistakes, unauthorized software, and compromised user accounts.

### 9. What is User Account Control?

User Account Control, or UAC, is a Windows security feature that helps control administrative elevation.

Even when administrative credentials are available, protected operations can require explicit elevation.

In my lab, I tested administrative elevation from the standard `finance.user` account using the authorized `labadmin` credentials.

### 10. How would you troubleshoot an access-denied error?

I would first determine what the user was attempting to do rather than immediately changing permissions.

I would verify the current security identity using:

```cmd
whoami
```

I would then inspect whether the operation requires administrative privileges.

If necessary, I would check local administrator group membership using:

```cmd
net localgroup administrators
```

If the user is correctly configured as a standard user and the operation legitimately requires administrative privileges, I would use an approved elevation process rather than permanently adding the user to the `Administrators` group.

### 11. What commands have you used to verify workstation and account configuration?

I have used commands including:

```cmd
hostname
whoami
net user
net localgroup administrators
```

| Command | Purpose |
|---|---|
| `hostname` | Verifies the computer name |
| `whoami` | Identifies the current security identity |
| `net user` | Lists local user accounts when run without additional arguments |
| `net localgroup administrators` | Displays membership of the local Administrators group |

### 12. Why should you avoid manually renaming a user profile folder?

The user profile directory is associated with Windows profile configuration and account information.

Simply renaming the folder can cause profile path inconsistencies and application or configuration problems.

If I need a clean account with a different username, I would generally create and configure the appropriate account and migrate required user data rather than manually renaming the profile directory.

### 13. Describe how you applied least privilege in your homelab.

In my Windows 11 enterprise homelab, I configured separate accounts for IT administration and normal employee use.

I created `labadmin` as a local administrator and `finance.user` as a standard Finance employee.

I verified the accounts and their group memberships using Windows administrative tools and command-line utilities.

I also tested a scenario where the standard user attempted an operation requiring administrative privileges.

Instead of permanently granting the employee administrator access, I used authorized administrator credentials for elevation.

This demonstrated the principle of least privilege and the separation of normal user activity from administrative operations.

---

## 2. Device Manager and Driver Troubleshooting

### 1. What is Device Manager?

Device Manager is a Windows administrative tool used to view and manage physical and virtual hardware devices and their associated drivers.

I can use it to inspect device status, driver information, hardware IDs, error codes, disabled devices, and unidentified devices.

### 2. What is the difference between a device and a driver?

A device is a physical or virtual hardware component.

A driver is software that allows Windows to communicate with and control that device.

Windows may detect that hardware exists while still being unable to use it correctly if the appropriate driver is unavailable.

### 3. What does a yellow warning indicator in Device Manager mean?

A yellow warning indicator means Windows has detected a problem associated with the device.

It does not automatically mean that the hardware has physically failed.

I would open the device Properties and inspect the Device status and error code before deciding on a corrective action.

### 4. What does Device Manager Code 28 mean?

Code 28 indicates that drivers for the device are not installed.

I encountered this in my Windows 11 VMware homelab when a VMware virtual device appeared as:

```text
Base System Device
```

With a yellow warning indicator.

### 5. How would you troubleshoot an unidentified device?

I would:

1. Inspect the device status.
2. Record the Device Manager error code.
3. Open **Properties → Details → Hardware Ids**.
4. Examine the vendor and device identifiers.
5. Identify the hardware.
6. Determine the appropriate trusted driver source.
7. Install the required driver or supporting software.
8. Restart if required.
9. Return to Device Manager and verify that the original warning is resolved.

### 6. What are hardware IDs, and how did you use them?

Hardware IDs are identifiers Windows uses to describe hardware devices.

For PCI devices, fields such as `VEN` and `DEV` can help identify the manufacturer and device.

During my lab, the unidentified device reported an ID beginning with:

```text
PCI\VEN_15AD&DEV_0740...
```

The vendor identifier:

```text
VEN_15AD
```

Identified VMware as the hardware vendor.

This helped narrow the investigation to a VMware virtual device.

### 7. Does an old driver date automatically mean the driver needs updating?

No. An old driver date by itself does not prove that the driver is causing a problem.

I would first consider:

- Device status
- Current symptoms
- Compatibility
- Recent changes
- Vendor recommendations
- Whether an approved newer driver is actually required

I would avoid unnecessary driver changes on a correctly functioning device.

### 8. When would you roll back a driver?

I would consider rolling back a driver when a device begins malfunctioning after a recent driver update and the previous driver was known to function correctly.

### 9. What is the difference between updating, rolling back, and reinstalling a driver?

| Action | Purpose |
|---|---|
| Updating | Installs a newer or more appropriate driver |
| Rolling back | Restores the previously installed driver and may help when a problem begins after an update |
| Reinstalling | Removes and installs the device or driver again and may help if the existing installation is corrupted |

I would diagnose the problem before deciding which action is appropriate.

### 10. Describe a device driver issue you resolved in your homelab.

During a Device Manager baseline inspection of my Windows 11 enterprise homelab, I discovered a `Base System Device` with a yellow warning indicator.

I inspected its Properties and found Device Manager Code 28, indicating that its driver was not installed.

Because Windows displayed only a generic device name, I inspected its hardware IDs. The identifier contained `VEN_15AD`, which identified VMware as the vendor.

I then checked the guest operating system and discovered that VMware Tools was not installed.

I installed VMware Tools and restarted the workstation.

After the restart, I reopened Device Manager and verified that the unidentified `Base System Device` and Code 28 warning were gone.

This taught me to use error codes and hardware identifiers to diagnose a device before making driver changes.

---

## 3. Task Manager and Performance Troubleshooting

### 1. How would you investigate a slow Windows workstation?

I would first clarify the symptoms, including when the slowdown occurs, whether it affects the whole system or one application, and whether anything recently changed.

I would then use Task Manager to inspect CPU, memory, disk, and network utilization.

If one resource shows unusually high or sustained utilization, I would identify the processes consuming it, determine whether the activity is expected, take the least disruptive appropriate corrective action, and verify performance afterward.

### 2. Which resources would you check first in Task Manager?

I would check overall:

- CPU
- Memory
- Disk
- Network

I would avoid immediately assuming that CPU, RAM, or another component is responsible.

I would use the resource data to determine which area deserves further investigation.

### 3. Does high CPU usage automatically identify the root cause?

No. High CPU usage is an observation rather than automatically the root cause.

Legitimate workloads such as applications, updates, security scans, or background processes can temporarily consume significant CPU resources.

I would determine whether the utilization is sustained and identify the responsible processes before taking corrective action.

### 4. What is the difference between the Processes and Performance tabs?

The **Processes** tab shows running applications and processes together with their individual CPU, memory, disk, and network utilization.

The **Performance** tab provides system-wide utilization information and historical graphs for resources such as CPU, memory, storage, and network interfaces.

I use Processes to identify what is consuming resources and Performance to understand overall system behavior and available resources.

### 5. What is a PID?

PID stands for Process Identifier.

Windows assigns a PID to each running process instance.

Multiple processes can use the same executable name while having different PIDs, so a PID helps distinguish a particular running process during troubleshooting.

### 6. Why does Microsoft Edge have multiple processes?

Modern browsers use a multi-process architecture.

Separate processes may handle:

- Browser tabs
- Rendering
- GPU processing
- Network services
- Extensions
- Subframes
- Storage
- Background functions

Seeing multiple Edge processes is therefore not automatically evidence of a problem.

### 7. Should you immediately end a process that is using substantial resources?

No. I would first determine what the application is doing and whether the user has unsaved work or an active session.

When practical, I would close the affected workload or application normally.

I would use forced termination when appropriate, such as when an application is unresponsive and cannot be closed normally.

### 8. How can startup applications affect performance?

Startup applications are programs configured to start automatically when a user signs into Windows.

Too many unnecessary startup applications can contribute to slower sign-in and additional resource consumption.

However, I would determine an application's purpose and business requirement before disabling it.

Security, VPN, backup, device-management, synchronization, virtualization, or other organizational applications may need to remain enabled.

### 9. What should you consider when troubleshooting performance inside a virtual machine?

I need to consider the resources allocated to the virtual machine rather than looking only at the physical host specifications.

For example, my Windows 11 lab workstation had:

- 8 GB RAM
- 2 virtual processors

A workload using a significant portion of those two virtual processors can noticeably affect the guest even though the physical host has a substantially more powerful processor.

### 10. Describe a performance issue you investigated in your homelab.

In my Windows 11 enterprise homelab, I investigated a simulated Finance workstation experiencing sluggish performance while Microsoft Edge was active.

Task Manager initially showed CPU utilization reaching approximately 96%. Subsequent observations remained around 50%, while memory was around 68–71% and disk and network activity were very low.

I sorted the Processes view by CPU and found Microsoft Edge repeatedly consuming approximately 38–44% CPU.

I expanded the Edge process group and identified individual browser content processes consuming a substantial portion of the processor resources. I also used the Details view to examine individual `msedge.exe` processes and PIDs.

I initially closed two high-resource browser tabs normally. CPU utilization remained elevated and another Edge tab became the main CPU consumer.

Because the first remediation did not resolve the symptom, I continued troubleshooting rather than assuming the problem was fixed.

After confirming that required user work would not be lost, I closed Microsoft Edge normally.

CPU utilization decreased to approximately 11%, while memory decreased to approximately 56%.

This supported the conclusion that the active browser workload accounted for most of the observed CPU pressure.

I also considered the VM's allocation of only two virtual processors when interpreting the performance data.

The experience reinforced the importance of measuring performance, testing a hypothesis, using the least disruptive corrective action, and verifying the result before closing an issue.

---

# 4. Windows Services

## 1. What is a Windows service?

A Windows service is a background component that provides operating system or application functionality without requiring direct user interaction.

Services can be managed through tools such as `services.msc`, PowerShell, and command-line utilities.

## 2. What is the difference between service status and startup type?

Service status describes the service's current state, such as Running or Stopped.

Startup type controls how the service is configured to start, such as Automatic, Manual, or Disabled.

For example, during my lab the Print Spooler was configured as Automatic, but I was able to stop it temporarily. Its startup configuration remained Automatic while its current status became Stopped.

## 3. How would you troubleshoot a Windows service?

I would first confirm the user's symptoms and determine whether the service is related to the problem.

I would check the service status, startup configuration, and dependencies before making changes. If appropriate, I could start or restart the service and then verify whether the original problem was resolved.

In my Windows 11 homelab, I practiced this with the Print Spooler and verified its stopped and running states using both `sc query spooler` and `Get-Service Spooler`.

---

# 5. Local Group Policy

## 1. What is Group Policy?

Group Policy is a Windows administration technology used to configure and enforce settings for computers and users.

I practiced with Local Group Policy on a Windows 11 workstation. Later, in an Active Directory environment, Group Policy can be centrally managed and applied across domain users and computers.

## 2. What is the difference between Computer Configuration and User Configuration?

Computer Configuration contains policies primarily applied to computers, while User Configuration contains policies primarily applied to users.

The appropriate area depends on whether the setting should follow the computer or the user.

## 3. What Group Policy tools have you used?

I have used `gpedit.msc` to configure Local Group Policy, `gpupdate /force` to request policy processing, `gpresult /r` to inspect Group Policy results, and `rsop.msc` to examine Resultant Set of Policy information.

In my homelab, I configured a security policy preventing Remote Desktop passwords from being saved.