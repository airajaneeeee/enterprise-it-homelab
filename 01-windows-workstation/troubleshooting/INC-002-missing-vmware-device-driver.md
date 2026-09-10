# INC-002 — VMware Virtual Device Driver Missing

## Incident Summary

During a baseline inspection of workstation `NS-W11-01`, Device Manager displayed a yellow warning indicator for a device identified as `Base System Device`.

## User Impact

No immediate end-user network outage was observed. However, Device Manager reported that a virtual hardware device was not configured correctly, requiring investigation before the workstation baseline could be considered complete.

## Initial Finding

Device Manager displayed:

- Device: `Base System Device`
- Manufacturer: Unknown
- Location: PCI bus 0, device 7, function 7
- Error: Code 28
- Status: Drivers for the device were not installed

## Investigation

The device Hardware IDs were inspected through:

`Device Manager → Base System Device → Properties → Details → Hardware Ids`

Primary Hardware ID:

`PCI\VEN_15AD&DEV_0740&SUBSYS_074015AD&REV_10`

The Compatible IDs and Device Instance Path were also reviewed.

The vendor identifier `VEN_15AD` identified VMware as the virtual hardware vendor.

Further investigation identified `DEV_0740` as a VMware VMCI device.

The guest operating system was checked for VMware Tools and VMware Tools was not installed.

## Preliminary Diagnosis

The Device Manager Code 28 warning is consistent with the VMware VMCI device driver being unavailable because VMware Tools has not yet been installed.

## Resolution

VMware Tools was installed in the Windows 11 guest operating system through VMware Workstation. The virtual machine was restarted after installation to complete the driver and integration component installation.

## Verification

After restarting `NS-W11-01`, Device Manager was inspected again.

The previously unidentified `Base System Device` was no longer listed under `Other devices`, and the previous Code 28 warning was no longer present.

This verified that Windows successfully recognized the VMware virtual device after the required VMware guest components and driver were installed.

## Root Cause

The Windows 11 guest operating system did not have the required VMware VMCI device driver installed because VMware Tools had not been installed.

Windows detected the VMware virtual hardware but could not associate it with the appropriate driver, causing the device to appear as `Base System Device` with Device Manager Code 28.

## Lessons Learned

- Device Manager error codes can help narrow the cause of hardware and driver problems.
- A yellow warning indicator should be investigated before making configuration changes.
- Hardware IDs can be used to identify devices that Windows cannot initially identify by name.
- PCI vendor and device identifiers can provide useful information during driver troubleshooting.
- Virtual machines require appropriate guest drivers for some virtual hardware.
- VMware Tools provides drivers and integration components for VMware guest operating systems.
- A troubleshooting change should always be followed by verification against the original problem.