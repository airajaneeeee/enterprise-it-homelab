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

## Local User Configuration

Two dedicated local accounts were configured for the workstation.

| Account | Purpose | Access Level |
|---|---|---|
| `labadmin` | IT administration | Local Administrator |
| `finance.user` | Simulated Finance employee | Standard User |

### Administrative Account

`labadmin` was added to the local `Administrators` group and is used for privileged IT administration tasks.

Verification commands:

```cmd
whoami
net localgroup administrators
```

### Standard Employee Account

`finance.user` was configured as a standard user and was not added to the local `Administrators` group.

This configuration supports the principle of least privilege by separating normal employee activity from administrative operations.

## Storage Configuration

| Volume | Purpose | File System | Capacity | Status |
|---|---|---|---|---|
| `C:` | Windows operating system | NTFS | 952.60 GB | Healthy |
| `Finance-Data (F:)` | Simulated Finance department data | NTFS | ~10 GB | Healthy |

A secondary virtual disk was provisioned and configured using Windows Disk Management. The disk was initialized, formatted using NTFS, assigned drive letter `F:`, and labeled `Finance-Data`.

Read/write access to the new volume was verified through Windows File Explorer.

## Endpoint Security Baseline

| Control | Assessment |
|---|---|
| Microsoft Defender Antivirus | Reviewed |
| Real-Time Protection | Reviewed |
| Windows Defender Firewall | Reviewed |
| User Account Control | Tested |
| Least-Privilege User Configuration | Implemented |
| BitLocker | Protection Off |

BitLocker status was assessed using:

`manage-bde -status`

The workstation was found to have BitLocker protection disabled. Encryption at rest has been identified as a future endpoint-hardening consideration pending evaluation of virtual TPM and recovery-key configuration.
