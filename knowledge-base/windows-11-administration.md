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