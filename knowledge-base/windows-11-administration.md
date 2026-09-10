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