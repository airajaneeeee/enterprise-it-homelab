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

The server is being prepared to host DNS for the future Active Directory environment.

Its preferred DNS server is therefore configured as:

`192.168.252.10`

which is `NS-DC01` itself.

Before the DNS Server role is installed, DNS queries to that address are expected to fail because no DNS service is listening there yet.

After DNS is deployed, DNS functionality will be validated again.

Domain clients should use the organization's Active Directory-aware internal DNS service rather than bypassing it with arbitrary public DNS resolvers.

---

# DNS vs Network Connectivity

Network connectivity and DNS resolution should be tested separately.

For example:

```cmd
ping 8.8.8.8
```

can test external IP connectivity without requiring DNS name resolution.

A DNS query can then be tested separately:

```cmd
nslookup microsoft.com
```

In the Northstar baseline:

```text
ping 8.8.8.8
→ SUCCESS

nslookup microsoft.com
→ FAILED
```

This demonstrated that external IP connectivity was available while DNS resolution was unavailable.

---

# Basic Network Troubleshooting Workflow

A useful initial troubleshooting sequence is:

1. Inspect the system's network configuration.
2. Verify connectivity to the local/default gateway.
3. Verify connectivity to an external IP address.
4. Test DNS name resolution.
5. Determine whether the failure is related to local configuration, routing/connectivity, or DNS.

Useful Windows commands include:

```cmd
ipconfig /all
ping <address>
nslookup <hostname>
```

Additional tools can be introduced depending on the problem.

---

# Production vs Homelab

The Northstar environment is designed to practice production-oriented concepts, but it is not presented as a production deployment.

Current homelab limitations include:

- Single physical host
- Single planned Domain Controller
- VMware NAT networking
- Limited compute resources
- No infrastructure redundancy yet

Production environments may additionally implement:

- Multiple Domain Controllers
- Redundant DNS services
- Dedicated VLANs/subnets
- Formal IP address management
- Monitoring and alerting
- Backup and disaster recovery
- Security baselines
- Privileged access controls
- Change management
- Capacity and availability planning

The purpose of the homelab is to implement applicable enterprise concepts while understanding where production architecture would differ.