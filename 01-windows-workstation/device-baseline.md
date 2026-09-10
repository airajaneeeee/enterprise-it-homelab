# Device and Driver Baseline — NS-W11-01

## Purpose

This document records the initial device and driver assessment performed on the Northstar Solutions Windows 11 workstation using Windows Device Manager.

## Network Adapter

| Property | Value |
|---|---|
| Device | Intel(R) 82574L Gigabit Network Connection |
| Driver Provider | Microsoft |
| Driver Date | 2015-08-03 |
| Driver Version | 12.19.1.32 |
| Digital Signer | Microsoft Windows |

## Device Manager Assessment

Device Manager was inspected for disabled devices, unknown devices, warning indicators, and potential driver problems.

During the assessment, a `Base System Device` was identified under `Other devices` with a yellow warning indicator.

## Base System Device Finding

| Property | Value |
|---|---|
| Device | Base System Device |
| Device Type | Other devices |
| Manufacturer | Unknown |
| Location | PCI bus 0, device 7, function 7 |
| Device Manager Error | Code 28 |
| Status | Drivers are not installed |

### Hardware Identification

The device hardware ID was inspected through:

`Device Manager → Base System Device → Properties → Details → Hardware Ids`

The primary hardware ID observed was:

`PCI\VEN_15AD&DEV_0740&SUBSYS_074015AD&REV_10`

### Initial Assessment

The `VEN_15AD` portion of the hardware ID identifies VMware as the PCI vendor.

No driver changes were made during the initial assessment. The device will be identified and an appropriate corrective action determined before modifying the system.

## Investigation

The unidentified device reported Device Manager error **Code 28**, indicating that its driver was not installed.

The device Hardware IDs and Compatible IDs were inspected to identify the virtual hardware.

Primary Hardware ID:

`PCI\VEN_15AD&DEV_0740&SUBSYS_074015AD&REV_10`

Compatible ID:

`PCI\VEN_15AD&DEV_0740&REV_10`

The PCI vendor identifier `VEN_15AD` corresponds to VMware. Further investigation identified `DEV_0740` as a VMware VMCI (Virtual Machine Communication Interface) device.

VMware Tools was also checked on the guest operating system and was not installed.

## Preliminary Diagnosis

The Base System Device warning is consistent with a missing VMware VMCI device driver in the Windows guest operating system.

VMware Tools has not yet been installed, so remediation will include installing VMware Tools and verifying whether the VMCI device is correctly recognized afterward.

## Remediation

VMware Tools was installed in the Windows 11 guest operating system to provide the appropriate VMware guest drivers and integration components.

The virtual machine was restarted after installation.

## Post-Remediation Verification

Device Manager was inspected after the restart.

The previously unidentified `Base System Device` and its Code 28 warning were no longer present.

The remediation successfully resolved the missing VMware virtual device driver condition.

## Final Status

**Resolved**

The workstation no longer displays the previously observed Base System Device warning in Device Manager.