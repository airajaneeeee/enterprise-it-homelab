# System Baseline — NS-DC01

## Purpose

This document records the pre-Active Directory system and network baseline for `NS-DC01`.

Baseline captured before deployment of Active Directory Domain Services and DNS Server.

## System Identity

| Item | Value |
|---|---|
| VMware VM Name | Northstar-DC01 |
| Windows Hostname | `NS-DC01` |
| Operating System | Windows Server 2025 Standard Evaluation |
| Windows Version | 2009 |
| OS Build | 26100 |
| Platform | VMware Workstation |
| Current Role | Standalone Windows Server |
| Planned Role | Domain Controller / DNS Server |

## Virtual Hardware

| Component | Configuration |
|---|---|
| Processor | 2 vCPU |
| Memory | 4 GB |
| Virtual Disk | ~60 GB |
| Network Mode | VMware NAT / VMnet8 |

## VMware Network

| Setting | Value |
|---|---|
| Virtual Network | VMnet8 |
| Network Address | `192.168.252.0/24` |
| Subnet Mask | `255.255.255.0` |
| NAT / Default Gateway | `192.168.252.2` |
| DHCP Range | `192.168.252.128-192.168.252.254` |

## Initial Network Configuration

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.129` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.254` |
| DNS Server | `192.168.252.2` |
| DNS Suffix | `localdomain` |
| DHCP Enabled | Yes |

## Static Network Configuration

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.10` |
| Subnet Mask | `255.255.255.0` |
| Prefix Length | `/24` |
| Default Gateway | `192.168.252.2` |
| Preferred DNS | `192.168.252.10` |
| Alternate DNS | None |
| DHCP Enabled | No |

## Validation

### Hostname

```cmd
hostname
```

Expected/verified hostname:

```text
NS-DC01
```

### Gateway Connectivity

```cmd
ping 192.168.252.2
```

**Result:** Successful

### External IP Connectivity

```cmd
ping 8.8.8.8
```

**Result:**

- Sent: 4
- Received: 4
- Lost: 0
- Packet loss: 0%

### Pre-DNS Resolution Test

```cmd
nslookup microsoft.com
```

Observed result:

```text
Server:  UnKnown
Address: 192.168.252.10

*** UnKnown can't find microsoft.com: No response from server
```

**Status:** Expected pre-deployment result.

`NS-DC01` is configured to use its own address as the preferred DNS server, but the DNS Server role has not yet been deployed.

This test will be repeated after DNS deployment.

## Baseline Status

**Windows Server Pre-AD Baseline — Complete**

Next infrastructure change:

- Install Active Directory Domain Services
- Install DNS Server
- Promote `NS-DC01` to a Domain Controller
- Deploy the Northstar Active Directory domain
- Perform post-deployment validation