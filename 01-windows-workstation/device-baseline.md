# Device and Driver Baseline — NS-W11-01

## Purpose

This document records the device and driver assessment performed on `NS-W11-01`, the Windows 11 Pro Finance workstation used in the Northstar Solutions enterprise IT homelab.

The assessment included reviewing installed hardware, inspecting driver information, identifying a missing VMware virtual-device driver, performing remediation, and validating the resulting device state.

## Network Adapter Baseline

The workstation network adapter and installed driver were inspected using Windows Device Manager.

| Property | Value |
|---|---|
| Device | Intel(R) 82574L Gigabit Network Connection |
| Driver Provider | Microsoft |
| Driver Date | 2015-08-03 |
| Driver Version | 12.19.1.32 |
| Digital Signer | Microsoft Windows |

The adapter was reviewed as part of the workstation device baseline before investigating devices reporting warnings or missing drivers.

## Device Manager Assessment

Windows Device Manager was inspected for:

- Unknown devices
- Disabled devices
- Warning indicators
- Missing drivers
- Device-status errors
- Driver information

During the assessment, a `Base System Device` was identified under **Other devices** with a yellow warning indicator.

## Device Finding

| Property | Value |
|---|---|
| Device | Base System Device |
| Device Type | Other devices |
| Manufacturer | Unknown |
| Location | PCI bus 0, device 7, function 7 |
| Device Manager Error | Code 28 |
| Initial Status | Drivers are not installed |

Device Manager **Code 28** indicated that Windows did not have an appropriate driver installed for the device.

## Hardware Identification

Rather than installing an unidentified driver immediately, the device Hardware IDs were inspected through:

**Device Manager → Base System Device → Properties → Details → Hardware Ids**

The primary Hardware ID observed was:

```text
PCI\VEN_15AD&DEV_0740&SUBSYS_074015AD&REV_10
```

A compatible identifier was also reviewed:

```text
PCI\VEN_15AD&DEV_0740&REV_10
```

### Hardware ID Interpretation

The identifiers provided information about the virtual hardware:

| Identifier | Interpretation |
|---|---|
| `VEN_15AD` | VMware PCI vendor |
| `DEV_0740` | VMware VMCI device |

The device was therefore identified as a VMware **VMCI (Virtual Machine Communication Interface)** device.

## Root Cause Assessment

The investigation also determined that VMware Tools was not installed in the Windows guest operating system.

Because VMware Tools provides guest drivers and integration components for VMware virtual machines, the missing VMware components were consistent with the unidentified VMCI device and Device Manager Code 28 condition.

### Root Cause

**Missing VMware guest driver components because VMware Tools was not installed.**

## Remediation

VMware Tools was installed in the Windows 11 guest operating system to provide the appropriate VMware guest drivers and integration components.

The virtual machine was restarted after installation so Windows could load and initialize the newly installed components.

## Post-Remediation Verification

After the restart, Device Manager was inspected again.

The previously unidentified `Base System Device` was no longer listed under **Other devices**, and the associated Code 28 warning was no longer present.

This confirmed that installation of VMware Tools successfully resolved the missing virtual-device driver condition.

## Troubleshooting Method

The device issue was handled using the following workflow:

1. Identified a Device Manager warning.
2. Reviewed the reported Device Manager error code.
3. Inspected the device Hardware IDs.
4. Used the PCI vendor and device identifiers to determine the hardware type.
5. Checked whether the required VMware guest components were installed.
6. Identified the likely root cause.
7. Installed VMware Tools.
8. Restarted the workstation.
9. Rechecked Device Manager.
10. Verified that the warning was resolved.

This approach avoided installing an arbitrary driver before identifying the affected hardware and likely cause.

## Production Considerations

In a production environment, driver remediation should follow organizational endpoint-management and change-control practices.

Before installing or updating drivers, an administrator should consider:

- Hardware and operating-system compatibility
- Approved software and driver sources
- Vendor documentation
- Driver signing and authenticity
- Business impact and restart requirements
- Change-management requirements
- Rollback or recovery options
- Post-change validation

Drivers and guest integration components should be obtained from approved and trusted sources rather than unverified third-party download sites.

## Final Status

**Resolved**

The unidentified VMware virtual device and Device Manager Code 28 condition were successfully resolved after VMware Tools was installed and the workstation was restarted.

`NS-W11-01` no longer displayed the previously observed `Base System Device` warning during post-remediation verification.