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