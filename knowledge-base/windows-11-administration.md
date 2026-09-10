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