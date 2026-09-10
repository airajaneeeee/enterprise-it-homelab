# System Baseline — NS-W11-01

## Purpose

This document records the baseline configuration of the Windows 11 workstation used in the Northstar Solutions enterprise IT homelab.

## System Identity

| Item | Value |
|---|---|
| VMware VM Name | Northstar-W11-Client01 |
| Windows Hostname | NS-W11-01 |
| Operating System | Windows 11 Pro |
| Windows Version | 25H2 |
| OS Build | 26200.9445 |
| Platform | VMware Workstation |
| Assigned Role | Finance Department Workstation |

## Virtual Hardware

| Component | Value |
|---|---|
| Processor / vCPU | 2 vCPUs — 12th Gen Intel(R) Core(TM) i7-12700H (2.69GHz) |
| Installed RAM | 8.00 GB |
| System Type | 64-bit operating system, x64-based processor |

## Configuration Change — Workstation Rename

### Original State

The workstation initially used the Windows computer name created during the original setup.

### Change

The workstation was renamed to:

`NS-W11-01`

### Naming Convention

- `NS` = Northstar Solutions
- `W11` = Windows 11
- `01` = Workstation 01

### Verification

The configuration was verified using:

```cmd
hostname
```

Expected result:

```text
NS-W11-01
```

The Windows **Settings > System > About** page was also reviewed to confirm the workstation hostname, operating system edition, Windows version, OS build, installed memory, processor information, and system architecture.