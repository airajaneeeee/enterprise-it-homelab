# Windows Server Administration — Knowledge Base

## Windows Server Roles and Features

### Server Role

A server role represents a primary function that a Windows Server performs for an organization.

Examples include:

- Active Directory Domain Services
- DNS Server
- DHCP Server
- File and Storage Services
- Web Server (IIS)

### Feature

A feature provides additional operating-system functionality or supports server roles.

Roles describe major server responsibilities, while features provide supporting capabilities.

---

## Server Naming

The Northstar Windows Server uses the hostname:

`NS-DC01`

Naming convention:

- `NS` — Northstar Solutions
- `DC` — Domain Controller
- `01` — first server assigned to the role

Consistent naming helps administrators identify the purpose of systems and simplifies documentation, monitoring, troubleshooting, and automation.

A naming convention should be documented and consistently applied rather than created independently for each server.

---

# IPv4 Addressing

## IP Address

An IPv4 address identifies an interface on an IPv4 network.

Northstar's server network is:

`192.168.252.0/24`

The server uses:

`192.168.252.10`

## Subnet Mask

The `/24` prefix corresponds to:

`255.255.255.0`

For this subnet:

- Network address: `192.168.252.0`
- Usable host range: addresses within the subnet excluding reserved network and broadcast addresses
- Broadcast address: `192.168.252.255`

## Default Gateway

The default gateway provides a route from the local subnet toward other networks.

Northstar VMware gateway:

`192.168.252.2`

If a destination is outside the local subnet, traffic can be sent toward the configured gateway.

---

# DHCP

DHCP stands for Dynamic Host Configuration Protocol.

DHCP can automatically provide clients with network configuration such as:

- IPv4 address
- Subnet mask
- Default gateway
- DNS server addresses
- Other DHCP options

Before static configuration, `NS-DC01` received:

`192.168.252.129`

from VMware's DHCP service.

The observed VMware DHCP allocation range is:

`192.168.252.128-192.168.252.254`

---

# Static Addressing

Core infrastructure systems commonly require predictable addressing.

`NS-DC01` was configured with:

`192.168.252.10`

This address is outside the configured VMware DHCP allocation range.

The lab uses manual static addressing for the server.

In production, addressing should be coordinated through the organization's network and IP address management practices. Depending on the design, administrators may use static assignments, DHCP exclusions, reservations, dedicated infrastructure ranges, or centralized IP address management.

---

# DNS

DNS stands for Domain Name System.

DNS translates names into information required to locate network resources.

For example, applications generally work with names such as:

`microsoft.com`

rather than requiring users to remember server IP addresses.

DNS is especially important in Microsoft Active Directory environments because domain clients use DNS to discover Domain Controllers and Active Directory services.

---

# Why NS-DC01 Uses Its Own DNS Address

`NS-DC01` hosts DNS for the Northstar Solutions Active Directory environment.

Its preferred DNS server is configured as:

`192.168.252.10`

which is the address of `NS-DC01` itself.

Active Directory relies heavily on DNS. Domain members use the internal DNS infrastructure to locate Domain Controllers and Active Directory services.

Domain clients should therefore use the organization's Active Directory-aware internal DNS infrastructure rather than bypassing it with arbitrary public DNS resolvers.

External DNS names can still be resolved through the internal DNS infrastructure.

---

# DNS vs Network Connectivity

Network connectivity and DNS resolution should be tested separately.

For example:

```cmd
ping 8.8.8.8
```

can test external IP connectivity without requiring DNS name resolution.

DNS resolution can then be tested separately using tools such as:

```cmd
nslookup microsoft.com
```

or:

```powershell
Resolve-DnsName microsoft.com
```

Before DNS Server was installed on `NS-DC01`, the server successfully reached an external IP address but could not obtain a DNS response from `192.168.252.10`.

After DNS Server deployment, external DNS resolution succeeded.

This demonstrated that IP connectivity and DNS resolution are separate functions that should be tested independently.

---

# Active Directory Domain Services

Active Directory Domain Services, or AD DS, provides centralized directory and identity services for Windows domain environments.

AD DS can centrally manage objects such as:

- Users
- Computers
- Groups
- Organizational Units
- Domain Controllers

It also supports authentication, authorization, Group Policy, and other centralized administrative capabilities.

In the Northstar Solutions lab, AD DS was installed on:

`NS-DC01`

The server was then promoted to a Domain Controller.

---

# Active Directory Forest

An Active Directory forest is the top-level logical structure of an Active Directory environment.

A forest can contain one or more domains that share important directory components such as the Active Directory schema and configuration.

The Northstar Solutions forest is:

`ad.northstarsolutions.com`

This is currently a single-domain forest.

---

# Active Directory Domain

A domain is a logical administrative and security boundary within Active Directory containing directory objects such as users, computers, and groups.

The Northstar Solutions Active Directory domain is:

`ad.northstarsolutions.com`

Its distinguished name is:

```text
DC=ad,DC=northstarsolutions,DC=com
```

The NetBIOS domain name is:

`NORTHSTAR`

---

# Domain Controller

A Domain Controller is a Windows Server running Active Directory Domain Services and providing directory services for an Active Directory domain.

Domain Controllers participate in functions such as:

- User and computer authentication
- Directory queries
- Group membership processing
- Group Policy processing
- Active Directory service discovery

Northstar's first Domain Controller is:

`NS-DC01.ad.northstarsolutions.com`

Because this is a resource-constrained homelab, only one Domain Controller currently exists.

A production environment may deploy multiple Domain Controllers to improve availability and redundancy.

---

# Domain Controller Promotion

Installing the Active Directory Domain Services role does not by itself make a Windows Server a Domain Controller.

After AD DS was installed on `NS-DC01`, the server was promoted to a Domain Controller.

During promotion, a new forest was created with the root domain:

`ad.northstarsolutions.com`

The server was also configured as:

- DNS Server
- Global Catalog

After promotion, the server restarted and began operating as a Domain Controller in the `NORTHSTAR` domain.

---

# DNS and Active Directory

DNS is a critical dependency for Active Directory.

Domain clients use DNS to discover Domain Controllers and Active Directory services.

The Northstar domain uses:

`ad.northstarsolutions.com`

The Domain Controller FQDN is:

`NS-DC01.ad.northstarsolutions.com`

The internal DNS server successfully resolves this name to:

`192.168.252.10`

Domain clients introduced later in the project will need to use the Northstar internal DNS infrastructure before joining and reliably operating in the domain.

---

# Internal vs External DNS Resolution

Internal DNS resolution refers to resolving names belonging to the organization's internal namespace.

Example:

```text
NS-DC01.ad.northstarsolutions.com
```

External DNS resolution refers to resolving names outside the internal namespace.

Example:

```text
microsoft.com
```

Both were tested after DNS deployment using:

```powershell
Resolve-DnsName NS-DC01.ad.northstarsolutions.com
Resolve-DnsName microsoft.com
```

The internal Domain Controller FQDN resolved to:

`192.168.252.10`

External DNS resolution also succeeded.

This demonstrated that the DNS infrastructure could resolve both the internal Active Directory namespace and external DNS names.

---

# Global Catalog

A Global Catalog is a Domain Controller capability that stores a searchable representation of objects in the Active Directory forest.

It supports important directory operations and forest-wide object searches.

During Domain Controller deployment, `NS-DC01` was configured as a Global Catalog.

The configuration was verified using:

```powershell
Get-ADForest
```

which identified:

`NS-DC01.ad.northstarsolutions.com`

as a Global Catalog.

---

# Forest and Domain Functional Levels

Active Directory forest and domain functional levels determine which Active Directory capabilities can be used and which Windows Server versions can participate as Domain Controllers.

The Northstar lab uses:

- Forest functional level: Windows Server 2025
- Domain functional level: Windows Server 2025

These were selected because the lab is being built entirely around Windows Server 2025 Domain Controller infrastructure.

Production organizations must evaluate compatibility with existing Domain Controllers and applications before changing functional levels.

---

# Directory Services Restore Mode

Directory Services Restore Mode, or DSRM, is a special recovery mode used for Active Directory maintenance and recovery operations.

A DSRM password was configured during Domain Controller promotion.

The password is intentionally not stored in the public project documentation.

Normal Domain Controller sign-in does not use the DSRM password.

---

# DNS Delegation Warning During Forest Creation

During creation of the new forest, the Domain Controller promotion wizard displayed a warning indicating that DNS delegation could not be created because an authoritative parent zone could not be found.

In this isolated homelab, the new Active Directory forest was being created without an existing authoritative parent DNS infrastructure managing the simulated namespace.

A DNS delegation was therefore not created.

The warning did not prevent successful creation of the new forest and DNS environment.

---

# Validating Active Directory

Infrastructure deployment should be validated after installation rather than assuming that completion of an installation wizard means every service is functioning correctly.

Useful PowerShell commands include:

```powershell
Get-ADDomain
Get-ADForest
Get-Service NTDS,DNS
```

In the Northstar lab, these commands confirmed:

- Correct Active Directory domain
- Correct forest root
- Correct NetBIOS domain
- Windows Server 2025 functional levels
- `NS-DC01` operating as the Domain Controller
- Global Catalog configuration
- Active Directory Domain Services running
- DNS Server running

---

# FSMO Roles — Current Observation

Active Directory uses Flexible Single Master Operations, or FSMO, roles for certain directory operations that require a single authoritative role holder.

Because `NS-DC01` is currently the only Domain Controller in the new Northstar forest, validation output showed it holding the FSMO roles returned by:

```powershell
Get-ADDomain
Get-ADForest
```

The observed roles included:

- PDC Emulator
- RID Master
- Infrastructure Master
- Schema Master
- Domain Naming Master

FSMO roles will be studied in greater detail later in the Windows Server module.

---

# Basic Network and Active Directory Troubleshooting Workflow

A useful initial troubleshooting sequence is:

1. Inspect the system's network configuration.
2. Verify the IPv4 address and subnet.
3. Verify connectivity to the default gateway.
4. Verify external IP connectivity if required.
5. Verify the configured DNS server.
6. Test internal DNS resolution.
7. Test external DNS resolution.
8. Verify relevant Active Directory and DNS services.
9. Determine whether the failure involves addressing, routing, DNS, directory services, or another component.
10. Remediate and retest the original symptom.

Useful commands include:

```cmd
ipconfig /all
ping <address>
nslookup <hostname>
```

and:

```powershell
Resolve-DnsName <hostname>
Get-Service NTDS,DNS
Get-ADDomain
Get-ADForest
```

---

# Production vs Homelab

The Northstar environment practices production-oriented concepts but is not presented as a production deployment.

Current homelab limitations include:

- Single physical host
- Single Domain Controller
- Single DNS Server
- VMware NAT networking
- Limited compute resources
- No infrastructure redundancy

Production environments may additionally implement:

- Multiple Domain Controllers
- Redundant DNS services
- Dedicated VLANs and subnets
- Formal IP address management
- Monitoring and alerting
- Active Directory backup and recovery
- Privileged administrative account separation
- Security baselines
- Centralized logging and auditing
- Change management
- Capacity and availability planning
- Disaster recovery

The goal of the Northstar homelab is to implement applicable enterprise administration concepts while understanding where a real production architecture would require additional controls and redundancy.