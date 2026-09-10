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