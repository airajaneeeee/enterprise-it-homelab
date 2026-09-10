# Windows 11 Administration — Knowledge Base

## Computer Name / Hostname

A hostname is the name used to identify a computer.

Windows may automatically assign a hostname during installation, but organizations often use standardized naming conventions to make computers easier to identify and manage.

### Command

```cmd
hostname
```

### Example

```text
NS-W11-01
```

---

## Current User Identity

The `whoami` command displays the current security identity.

### Command

```cmd
whoami
```

### Example

```text
NS-W11-01\labadmin
```

In this example:

- `NS-W11-01` is the computer.
- `labadmin` is the user account.

---

## VMware VM Name vs Windows Hostname

The VMware VM name and the Windows hostname are separate.

The VMware name identifies the virtual machine inside the hypervisor.

The Windows hostname identifies the Windows computer to the operating system and network.

Example:

- VMware VM Name: `Northstar-W11-Client01`
- Windows Hostname: `NS-W11-01`

---

## Troubleshooting Scenario — Hostname Changed but Username Did Not

### Situation

After renaming the computer, the hostname changed but the existing username remained unchanged.

### Example

Before:

```text
DESKTOP-12345\aira
```

After:

```text
NS-W11-01\aira
```

### Explanation

The computer name and user account are separate.

Renaming the Windows computer changes the hostname but does not rename existing user accounts.

### Resolution

No action is required if the user account is still valid.

If a differently named account is required, create a new properly configured account instead of manually renaming the user profile folder.

---

## Important — Do Not Manually Rename User Profile Folders

Suppose you see this user profile folder:

```text
C:\Users\oldname
```

Do **not** manually rename the folder.

Windows stores references to user profiles in multiple places. Manually changing the profile directory can lead to profile loading problems and broken paths.

Instead, a safer approach is to create and verify a correctly configured replacement account. We will create a new lab administrator account with the intended username and let Windows create its user profile folder.

---

## Local Accounts

A local account is stored and authenticated on an individual Windows computer.

Example:

```text
NS-W11-01\finance.user
```

This account exists on `NS-W11-01` and is not currently managed through Active Directory.

---

## Administrator vs Standard User

### Administrator

An administrator can perform privileged system operations such as:

- Installing software
- Managing users
- Changing security settings
- Managing services
- Modifying protected system configuration

### Standard User

A standard user can perform normal day-to-day tasks but has limited ability to make system-wide changes.

---

## Principle of Least Privilege

The principle of least privilege means that users and systems should receive only the permissions required for their responsibilities.

In this lab:

- `labadmin` is used for IT administrative operations.
- `finance.user` is used for standard Finance employee activity.

This reduces unnecessary administrative access.

---

## User Account Control

User Account Control, or UAC, helps control privileged changes in Windows.

When a standard user attempts an administrative operation, Windows can request approved administrator credentials.

This allows administrative work to be authorized without permanently making the user an administrator.

---

## Useful Commands

### Display Local Users

```cmd
net user
```

### Display Current Identity

```cmd
whoami
```

### Display Local Administrators

```cmd
net localgroup administrators
```

---

## Troubleshooting Scenario — Access Denied

### Symptom

A command returns:

```text
System error 5 has occurred.
Access is denied.
```

### Possible Cause

The command may require administrator privileges.

### Troubleshooting

1. Confirm whether the operation requires elevation.
2. Confirm that the user is authorized to perform the task.
3. If appropriate, use an approved administrator account or `Run as administrator`.
4. Retry the operation.
5. Verify the result.

### Lesson

An access-denied error is not automatically a software failure. It can be an intentional security control.

---

## Device Manager and Device Drivers

### What is Device Manager?

Device Manager is a Windows administrative tool used to view and manage physical and virtual hardware devices recognized by the operating system.

It can be used to:

- Check whether Windows recognizes a device
- View device status and error codes
- Inspect installed drivers
- View driver provider, version, date, and digital signer
- Enable or disable devices
- Update, roll back, or uninstall drivers
- Inspect hardware identifiers
- Troubleshoot unknown or malfunctioning devices

### Device vs Driver

A **device** is a physical or virtual hardware component available to the operating system.

A **driver** is software that allows Windows to communicate with and control that device.

A device can therefore be detected by Windows while still being unable to function correctly if the appropriate driver is unavailable.

### Device Manager Warning Indicators

A yellow warning indicator means Windows has detected a problem associated with the device.

It does not automatically mean that the hardware has physically failed.

The next troubleshooting step should be to inspect:

`Device → Properties → General → Device status`

The Device status can provide an error code and additional information about the problem.

### Device Manager Code 28

Code 28 indicates that drivers for the device are not installed.

Example encountered in the Northstar lab:

`The drivers for this device are not installed. (Code 28)`

A suitable troubleshooting process is:

1. Inspect the device status and error code.
2. Determine what device Windows is detecting.
3. Inspect its Hardware IDs if the device is unidentified.
4. Identify the manufacturer and device.
5. Determine the appropriate trusted driver source.
6. Install the appropriate driver or supporting software.
7. Restart if required.
8. Return to Device Manager and verify that the original warning is resolved.

### Hardware IDs

Hardware IDs can help identify hardware when Windows displays a generic device name such as `Base System Device`.

They can be found through:

`Device Manager → Device Properties → Details → Hardware Ids`

Example from the lab:

`PCI\VEN_15AD&DEV_0740&SUBSYS_074015AD&REV_10`

Important PCI identifiers include:

- `VEN` — Vendor identifier
- `DEV` — Device identifier
- `SUBSYS` — Subsystem identifier
- `REV` — Hardware revision

In this lab:

`VEN_15AD`

identified VMware as the PCI device vendor.

The device identifier:

`DEV_0740`

was associated with the VMware VMCI virtual device.

### Compatible IDs

Compatible IDs provide Windows with additional identifiers that can be used when matching a device with an appropriate driver.

During the investigation, the device reported compatible identifiers including:

`PCI\VEN_15AD&DEV_0740&REV_10`

and more general PCI class identifiers.

Hardware IDs are generally more specific, while compatible IDs can provide broader matching information.

### Device Instance Path

The Device Instance Path uniquely identifies a particular device instance recognized by Windows.

The affected virtual device reported a path beginning with:

`PCI\VEN_15AD&DEV_0740...`

This provided additional confirmation that the unidentified device was the same VMware PCI device being investigated.

### VMware Tools

VMware Tools is a collection of guest operating system components and drivers that improves integration between a VMware virtual machine and the virtualization platform.

During this lab, VMware Tools was not initially installed on the Windows 11 guest.

This was relevant because Device Manager detected VMware virtual hardware but did not have the appropriate driver for one of the devices.

After VMware Tools was installed and the virtual machine restarted, Windows successfully recognized the previously unidentified device.

### Driver Date Does Not Automatically Mean a Driver Is Outdated

The network adapter in this lab reported:

- Device: `Intel(R) 82574L Gigabit Network Connection`
- Driver Provider: Microsoft
- Driver Date: 2015-08-03
- Driver Version: 12.19.1.32
- Digital Signer: Microsoft Windows

An old driver date alone is not sufficient evidence that a driver should be replaced.

Before changing a working driver, consider:

- Whether the device is functioning correctly
- Device Manager status
- Current symptoms
- Compatibility
- Recent changes
- Vendor recommendations
- Whether an approved newer driver is actually required

Updating drivers without a reason can introduce unnecessary problems.

### Driver Update vs Rollback vs Reinstallation

**Update Driver**

Used when a newer or appropriate driver is required to resolve compatibility, functionality, security, or support issues.

**Roll Back Driver**

Can be appropriate when a device begins malfunctioning after a driver update and the previous driver was known to work correctly.

**Uninstall Device / Reinstall Driver**

Can be considered when a driver installation is corrupted or other appropriate troubleshooting has failed.

This is generally more disruptive than simply inspecting the device or enabling a disabled device, so it should not automatically be the first troubleshooting step.

### Troubleshooting Principle

Avoid making changes before collecting evidence.

A useful troubleshooting sequence is:

`Observe → Gather Evidence → Identify → Form a Hypothesis → Remediate → Verify → Document`

The original symptom should always be tested again after remediation.

### Scenario — Base System Device Reports Code 28

**Environment:**  
Windows 11 Pro virtual workstation `NS-W11-01` running in VMware Workstation.

**Symptom:**  
During a Device Manager baseline inspection, a yellow warning indicator was discovered under `Other devices` for a device identified only as `Base System Device`.

**Investigation:**  
The device Properties window reported Device Manager Code 28, indicating that the required driver was not installed.

The device Hardware IDs, Compatible IDs, and Device Instance Path were inspected.

The primary Hardware ID was:

`PCI\VEN_15AD&DEV_0740&SUBSYS_074015AD&REV_10`

The vendor identifier `VEN_15AD` identified VMware as the virtual hardware vendor.

The guest operating system was then checked for VMware Tools, and VMware Tools was not installed.

**Root Cause:**  
The Windows guest did not have the required VMware VMCI device driver because VMware Tools had not been installed. Windows could detect the virtual hardware but could not associate it with the appropriate driver.

**Resolution:**  
VMware Tools was installed in the Windows 11 guest operating system and the virtual machine was restarted.

**Verification:**  
Device Manager was reopened after the restart. The previous `Base System Device` entry and Code 28 warning were no longer present.

**Result:**  
Resolved.

**Lessons Learned:**

- Device Manager error codes can significantly narrow a troubleshooting investigation.
- Unknown devices can be investigated using their Hardware IDs.
- PCI vendor and device identifiers can help determine the manufacturer and type of an unidentified device.
- Virtual machines still depend on appropriate device drivers.
- Driver changes should be based on evidence rather than performed randomly.
- Successful installation is not sufficient verification; the original symptom must be checked again afterward.

### Windows Tools Used

- Device Manager (`devmgmt.msc`)
- Device Properties
- Driver information
- Device status and error codes
- Hardware IDs
- Compatible IDs
- Device Instance Path
- VMware Tools

## Task Manager and Performance Troubleshooting

### What is Task Manager?

Task Manager is a Windows administrative and troubleshooting tool used to monitor running applications, processes, system resource utilization, startup applications, users, services, and overall system performance.

It can be opened with:

`Ctrl + Shift + Esc`

Task Manager is useful when investigating symptoms such as:

- Slow workstation performance
- High CPU utilization
- High memory utilization
- Excessive disk activity
- Unexpected network activity
- Unresponsive applications
- Excessive startup applications
- Suspicious or unfamiliar processes

### Resource Utilization

The Processes and Performance views provide information about several important system resources.

#### CPU

CPU utilization represents how much processor capacity is currently being used.

High CPU utilization can be caused by:

- Resource-intensive applications
- Background processes
- Browser tabs or web content
- Updates
- Security scans
- Software problems
- Processes that are not responding correctly

High CPU utilization alone does not identify the root cause.

The responsible process and the duration of the utilization should be investigated.

#### Memory

Memory utilization represents the amount of RAM being used by Windows, applications, and background processes.

High memory utilization may result from:

- Many applications running simultaneously
- Memory-intensive applications
- Insufficient physical or virtual RAM
- Background services
- Memory leaks

Memory percentage should be interpreted together with the total installed or assigned RAM and the amount of available memory.

#### Disk

Disk utilization represents storage activity.

High disk activity may occur during:

- File transfers
- Windows updates
- Software installation
- Search indexing
- Antivirus scanning
- Application I/O

#### Network

Network utilization represents network traffic generated by applications and system processes.

It can help identify applications performing downloads, uploads, synchronization, updates, or other network activity.

### Processes Tab

The Processes tab provides an application-oriented view of currently running processes and their resource consumption.

Processes can be sorted by:

- CPU
- Memory
- Disk
- Network

Sorting by a resource is useful when determining which application or process is contributing most to a performance problem.

A high resource value should be treated as evidence requiring investigation rather than automatic proof that the application is defective.

### Performance Tab

The Performance tab provides system-wide information and historical graphs for resources such as:

- CPU
- Memory
- Disk
- Ethernet/network interfaces

During the Northstar lab, the Windows 11 virtual workstation had:

- 8 GB RAM
- 2 virtual processors

The virtual machine configuration was important when interpreting performance because the guest operating system had access only to the resources allocated by the virtualization platform.

### Processes and Virtual Machines

Performance measurements inside a virtual machine must be interpreted according to the resources assigned to the guest.

A physical host may contain a powerful processor and large amount of memory while a virtual machine is configured with significantly fewer resources.

For example, `NS-W11-01` had only two virtual processors assigned.

A workload consuming a significant portion of those virtual processors may therefore have a noticeable impact on the guest even though the physical host has considerably more processing capacity.

### Details Tab and Process IDs

The Details tab provides a lower-level view of individual running processes.

A Process Identifier (PID) is a number assigned by Windows to a running process instance.

Multiple processes may use the same executable name while having different PIDs.

Example:

`msedge.exe`

Microsoft Edge uses a multi-process architecture, so multiple `msedge.exe` processes may run simultaneously.

Different processes may be responsible for:

- Browser tabs
- Rendering
- GPU processing
- Network services
- Storage services
- Extensions
- Background browser functions

PIDs help distinguish individual process instances during troubleshooting.

### Multi-Process Applications

Seeing many processes associated with one application does not automatically indicate a problem.

Modern applications such as web browsers intentionally separate workloads into multiple processes for stability, security, and performance.

During the lab, expanding Microsoft Edge in Task Manager revealed separate processes associated with browser tabs, subframes, GPU processing, network services, storage services, and other browser components.

This allowed application-level CPU usage to be narrowed to individual browser workloads.

### Startup Apps

The Startup apps section shows applications configured to start automatically when a user signs into Windows.

Important information includes:

- Application name
- Publisher
- Enabled or disabled status
- Startup impact

Excessive or unnecessary startup applications can contribute to slow sign-in or increased resource usage.

However, startup applications should not be disabled simply because they consume resources.

Some startup applications may be required for:

- Endpoint security
- VPN connectivity
- Device management
- Backup
- Cloud synchronization
- Collaboration tools
- Virtualization integration
- Organizational applications

The purpose and business requirement of an application should be understood before changing its startup configuration.

### Last BIOS Time

Task Manager may display a `Last BIOS time` value in Startup apps.

This represents time spent during firmware initialization before Windows startup.

It does not represent the complete Windows boot or user sign-in experience.

Therefore, a low Last BIOS time does not necessarily mean that a user's overall startup experience is fast.

### Efficiency Mode

Task Manager may show an efficiency indicator for certain processes.

Efficiency mode allows Windows to reduce the resource priority of eligible processes to improve power efficiency and responsiveness for higher-priority workloads.

The presence of an efficiency indicator does not automatically mean that the process is malfunctioning.

### Graceful Application Closure vs End Task

When resolving a performance issue, terminating a process should not automatically be the first action.

If possible, an application should be closed normally before using a forced termination method.

Graceful closure can help protect:

- Unsaved work
- Active sessions
- Open forms
- Application state
- User data

`End task` may be appropriate when an application is unresponsive or cannot be closed normally, but the potential user impact should be considered first.

### Performance Troubleshooting Method

A useful workflow for workstation performance troubleshooting is:

`Confirm Symptom → Establish Baseline → Identify Resource → Identify Process → Investigate Process → Form Hypothesis → Remediate → Verify → Document`

For example:

`Workstation Slow → CPU High → Sort by CPU → Identify Application → Inspect Child Processes → Test Corrective Action → Measure CPU Again`

### Scenario — High CPU Utilization During Edge Usage

During the Northstar lab, `NS-W11-01` demonstrated sustained CPU utilization while Microsoft Edge was active.

Task Manager showed CPU utilization reaching approximately 96% during an initial observation and remaining around 50% during subsequent observations.

Microsoft Edge repeatedly consumed approximately 38–44% CPU.

Expanding the Edge process group revealed individual browser content processes consuming a substantial portion of the CPU.

Two high-resource tabs were initially closed normally. CPU utilization remained elevated, demonstrating that the first remediation had not fully resolved the problem.

Microsoft Edge was then closed normally after confirming that required user work would not be lost.

After the application was closed:

- CPU utilization decreased to approximately 11%
- Memory utilization decreased to approximately 56%
- Disk utilization remained near 0%
- Network utilization remained near 0%

This supported the conclusion that the active Edge browsing workload accounted for most of the observed CPU pressure.

The investigation did not provide sufficient evidence to conclude that Microsoft Edge itself was defective.

### Lessons Learned

- High resource utilization is a symptom and requires further investigation.
- A single Task Manager snapshot may not represent sustained system behavior.
- Repeated measurements provide stronger evidence.
- The highest-consuming process should be investigated before corrective action is taken.
- Multiple processes can belong to one application.
- Virtual machine resource allocation affects how performance data should be interpreted.
- The least disruptive corrective action should generally be attempted first.
- A failed first remediation is useful diagnostic evidence.
- A troubleshooting hypothesis should be revised when the evidence does not support it.
- The original symptom must be retested after remediation.

### Windows Tools Used
- Task Manager (`taskmgr.exe`)
- Processes
- Performance monitoring
- Startup apps
- Details and Process IDs (PIDs)

## Windows Services

Windows services are background components that provide operating system and application functionality without requiring direct user interaction.

Services can be managed through the Services console:

`services.msc`

Common startup types include:

- **Automatic** — starts automatically with Windows.
- **Automatic (Delayed Start)** — starts automatically after the initial boot process.
- **Manual** — can be started when required by Windows, an application, or an administrator.
- **Disabled** — cannot start until its configuration is changed.

The startup type and current service status are different concepts. For example, a service configured as `Automatic` can currently be `Stopped`.

### Print Spooler Lab

I inspected and administered the Windows Print Spooler (`Spooler`) service.

The service was configured as:

- Startup type: `Automatic`
- Initial status: `Running`
- Executable: `C:\Windows\System32\spoolsv.exe`

I stopped and started the service and verified both states using Command Prompt and PowerShell.

Commands used:

`sc query spooler`

`Get-Service Spooler`

Before changing a service, its purpose, dependencies, and potential business impact should be understood.

## Local Group Policy

Local Group Policy allows administrators to configure and enforce Windows settings on an individual computer.

The Local Group Policy Editor can be opened with:

`gpedit.msc`

Two major policy areas are:

- **Computer Configuration** — policies primarily affecting the computer.
- **User Configuration** — policies primarily affecting users.

Policy settings commonly have three states:

- **Not Configured**
- **Enabled**
- **Disabled**

The meaning depends on the wording of the policy. For example, enabling `Do not allow passwords to be saved` enforces the restriction against saving Remote Desktop credentials.

### RDP Credential Security Lab

On `NS-W11-01`, I configured:

`Computer Configuration → Administrative Templates → Windows Components → Remote Desktop Services → Remote Desktop Connection Client → Do not allow passwords to be saved`

The policy was set to `Enabled`.

I also practiced using:

- `gpupdate /force` — refresh Group Policy processing
- `gpresult /r` — display Group Policy result information
- `rsop.msc` — inspect Resultant Set of Policy information

This provided hands-on experience configuring and investigating local Windows policy.

## Windows Network Troubleshooting Basics

### DNS Name Resolution and DNS Cache

DNS (Domain Name System) translates hostnames and domain names into IP addresses that computers use for network communication.

Useful Windows DNS troubleshooting commands include:

`nslookup <hostname>`

Tests DNS name resolution and displays the IP address returned for a hostname.

`ipconfig /displaydns`

Displays records currently stored in the Windows DNS resolver cache.

`ipconfig /flushdns`

Clears the Windows DNS resolver cache.

Flushing the DNS cache can be useful when a workstation contains stale or incorrect cached DNS information, such as after a server or website's DNS record has changed.

It should not be treated as a generic solution for all network connectivity problems.

### Network Configuration

`ipconfig /all` displays detailed TCP/IP configuration, including:

- IPv4 address
- Subnet mask
- Default gateway
- DHCP status
- DHCP server
- DNS servers
- DNS suffix
- Network adapter information

On the `NS-W11-01` VMware workstation, I used `ipconfig /all` to inspect the DHCP-assigned network configuration and identify the workstation's IPv4 address, default gateway, DHCP server, and DNS server.

### DHCP Release and Renew

DHCP (Dynamic Host Configuration Protocol) can automatically provide network configuration to client devices.

Useful commands include:

`ipconfig /release`

Releases the current DHCP-assigned IPv4 configuration.

`ipconfig /renew`

Requests network configuration from the DHCP server again.

A renewed DHCP lease may receive the same IP address as before. Renewal does not guarantee that the client will receive a different address.

Release and renew can be useful when troubleshooting DHCP-related configuration problems but should not be used blindly for every connectivity issue.

### DNS vs DHCP Troubleshooting

DNS and DHCP perform different functions.

- **DHCP** provides network configuration such as IP addressing information.
- **DNS** resolves names into IP addresses.

Therefore:

`ipconfig /release` and `ipconfig /renew` relate primarily to DHCP configuration.

`ipconfig /flushdns` relates to the local DNS resolver cache.

`nslookup` can be used to investigate DNS name resolution.

Understanding which network function is failing is more important than simply running a collection of troubleshooting commands.