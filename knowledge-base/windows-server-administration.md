# Windows Server Administration — Knowledge Base

This knowledge base explains concepts practiced through the Phase 5 domain-workstation checkpoint. Lab examples refer to the recorded Northstar environment; planned capabilities are identified separately.

For exact settings and evidence, see the [server baseline](../02-windows-server/server-baseline.md) and [screenshot guide](../02-windows-server/screenshots/README.md).

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

## IPv4 Addressing

### IP Address

An IPv4 address identifies an interface on an IPv4 network.

Northstar's server network is:

`192.168.252.0/24`

The server uses:

`192.168.252.10`

### Subnet Mask

The `/24` prefix corresponds to:

`255.255.255.0`

For this subnet:

- Network address: `192.168.252.0`
- Usable host range: addresses within the subnet excluding reserved network and broadcast addresses
- Broadcast address: `192.168.252.255`

### Default Gateway

The default gateway provides a route from the local subnet toward other networks.

Northstar VMware gateway:

`192.168.252.2`

If a destination is outside the local subnet, traffic can be sent toward the configured gateway.

---

## DHCP

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

## Static Addressing

Core infrastructure systems commonly require predictable addressing.

`NS-DC01` was configured with:

`192.168.252.10`

This address is outside the configured VMware DHCP allocation range.

The lab uses manual static addressing for the server.

In production, addressing should be coordinated through the organization's network and IP address management practices. Depending on the design, administrators may use static assignments, DHCP exclusions, reservations, dedicated infrastructure ranges, or centralized IP address management.

---

## DNS

DNS stands for Domain Name System.

DNS translates names into information required to locate network resources.

For example, applications generally work with names such as:

`microsoft.com`

rather than requiring users to remember server IP addresses.

DNS is especially important in Microsoft Active Directory environments because domain clients use DNS to discover Domain Controllers and Active Directory services.

---

## Why NS-DC01 Uses Its Own DNS Service

`NS-DC01` hosts DNS for the Northstar Solutions Active Directory environment.

Its current IPv4 DNS client setting is:

`127.0.0.1`

which is the IPv4 loopback address and points back to the DNS service running on `NS-DC01`.

The server's network IPv4 address remains:

`192.168.252.10`

Active Directory relies heavily on DNS.

Domain members use the internal DNS infrastructure to locate Domain Controllers and Active Directory services.

Domain clients should therefore use the organization's Active Directory-aware internal DNS infrastructure rather than bypassing it with arbitrary public DNS resolvers.

External DNS names can still be resolved through the internal DNS infrastructure.

---

## DNS Client Address vs DNS Forwarder

These are two different configurations.

A **DNS client address** tells a computer which DNS server to query.

For `NS-DC01`, the current IPv4 DNS client address is:

`127.0.0.1`

This points the Domain Controller back to the DNS Server service running locally.

A **DNS forwarder** is configured on the DNS Server service and tells that DNS server where to send queries that it cannot answer from its own authoritative zones or cache.

Northstar currently has no explicit DNS forwarders configured.

External DNS resolution still succeeds because Windows DNS can use root hints when no forwarder is available.

---

## Loopback Addresses

Loopback addresses refer back to the same computer.

Common examples:

- IPv4 loopback: `127.0.0.1`
- IPv6 loopback: `::1`

During `nslookup` testing on `NS-DC01`, the DNS server was displayed as:

`::1`

This still referred to the local server.

The presence of `::1` alone is not a reason to disable IPv6.

---

## IPv6 Baseline

IPv6 remains enabled on `NS-DC01`.

During the post-deployment baseline check:

- IPv4 connectivity reported Internet access.
- IPv6 connectivity reported `NoTraffic`.

The lab currently relies primarily on IPv4, but IPv6 was not disabled merely because the current workload is IPv4-based.

---

## DNS with Multiple Domain Controllers

The same core concept applies when an Active Directory environment has multiple Domain Controllers: domain systems should use internal Active Directory-aware DNS servers.

For example, with two Domain Controllers that both host DNS, each DC can use internal DNS and clients can be configured with more than one internal DNS server for redundancy.

Active Directory-integrated DNS zones can replicate through Active Directory.

This avoids maintaining independent manual copies of the same internal DNS zone.

A multi-DC design improves availability compared with Northstar's current single-DC lab, but the current environment intentionally remains small because of host resource constraints.

---

## Remote Administration and Firewall Baseline

Before beginning centralized Active Directory administration, `NS-DC01` was checked for basic administration readiness.

Verified items included:

- Network profile: `DomainAuthenticated`
- Remote Desktop: enabled
- Network Level Authentication: enabled
- Remote Management: enabled
- Windows Defender Firewall: enabled
- ICMPv4 Echo Request inbound rule: enabled for troubleshooting
- VMware Tools: operational
- Time zone: verified
- Windows Update: verified

The firewall was kept enabled.

Connectivity testing was supported by enabling the specific ICMPv4 rule rather than disabling firewall protection.

---

## DNS vs Network Connectivity

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

## Active Directory Domain Services

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

## Active Directory Forest

An Active Directory forest is the top-level logical structure of an Active Directory environment.

A forest can contain one or more domains that share important directory components such as the Active Directory schema and configuration.

The Northstar Solutions forest is:

`ad.northstarsolutions.com`

This is currently a single-domain forest.

---

## Active Directory Domain

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

## Domain Controller

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

## Domain Controller Promotion

Installing the Active Directory Domain Services role does not by itself make a Windows Server a Domain Controller.

After AD DS was installed on `NS-DC01`, the server was promoted to a Domain Controller.

During promotion, a new forest was created with the root domain:

`ad.northstarsolutions.com`

The server was also configured as:

- DNS Server
- Global Catalog

After promotion, the server restarted and began operating as a Domain Controller in the `NORTHSTAR` domain.

---

## DNS and Active Directory

DNS is a critical dependency for Active Directory.

Domain clients use DNS to discover Domain Controllers and Active Directory services.

The Northstar domain uses:

`ad.northstarsolutions.com`

The Domain Controller FQDN is:

`NS-DC01.ad.northstarsolutions.com`

The internal DNS server successfully resolves this name to:

`192.168.252.10`

`NS-W11-01` now uses `192.168.252.10` for DNS and is joined to the Northstar domain. Its pre-join validation demonstrated why IP connectivity alone is insufficient for Active Directory service discovery.

---

## Internal vs External DNS Resolution

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

## Global Catalog

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

## Forest and Domain Functional Levels

Active Directory forest and domain functional levels determine which Active Directory capabilities can be used and which Windows Server versions can participate as Domain Controllers.

The Northstar lab uses:

- Forest functional level: Windows Server 2025
- Domain functional level: Windows Server 2025

These were selected because the lab is being built entirely around Windows Server 2025 Domain Controller infrastructure.

Production organizations must evaluate compatibility with existing Domain Controllers and applications before changing functional levels.

---

## Directory Services Restore Mode

Directory Services Restore Mode, or DSRM, is a special recovery mode used for Active Directory maintenance and recovery operations.

A DSRM password was configured during Domain Controller promotion.

The password is intentionally not stored in the public project documentation.

Normal Domain Controller sign-in does not use the DSRM password.

---

## DNS Delegation Warning During Forest Creation

During creation of the new forest, the Domain Controller promotion wizard displayed a warning indicating that DNS delegation could not be created because an authoritative parent zone could not be found.

In this isolated homelab, the new Active Directory forest was being created without an existing authoritative parent DNS infrastructure managing the simulated namespace.

A DNS delegation was therefore not created.

The warning did not prevent successful creation of the new forest and DNS environment.

---

## Validating Active Directory

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

## FSMO Roles — Current Observation

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

## Basic Network and Active Directory Troubleshooting Workflow

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

## DNS Resource Records

Resource records describe the names, addresses, and services in a DNS zone.

| Record | Purpose | Northstar Example or Scope |
|---|---|---|
| A | Maps a hostname to an IPv4 address | `ns-dc01.ad.northstarsolutions.com` → `192.168.252.10` |
| AAAA | Maps a hostname to an IPv6 address | Concept reviewed; no unnecessary training record retained |
| CNAME | Maps an alias to another DNS name | Temporary `dns-alias` → `dns-lab` exercise |
| PTR | Maps a reverse DNS name to a hostname | `10.252.168.192.in-addr.arpa` → `ns-dc01.ad.northstarsolutions.com` |
| MX | Identifies a domain's mail exchanger | Concept reviewed; no mail server implemented |
| SRV | Identifies a service's target host and port, with priority and weight | AD service discovery |

### Temporary A and CNAME Records

The recorded administration exercise used this relationship:

```text
dns-alias.ad.northstarsolutions.com       CNAME
                  ↓
dns-lab.ad.northstarsolutions.com         A
                  ↓
192.168.252.10
```

A CNAME points to another DNS name rather than directly to an IP address. A successful alias lookup tests name resolution; it does not prove that an application service is available at the destination.

Both temporary records were removed after validation according to the lab notes. They do not represent additional permanent Northstar servers.

### SRV Records and Active Directory

Active Directory clients use SRV records to locate services. The inspected records included:

| Service | Observed TCP Port | Purpose |
|---|---:|---|
| LDAP | 389 | Directory access |
| Kerberos | 88 | Authentication |
| Kerberos password service | 464 | Password operations |
| Global Catalog | 3268 | Forest-wide directory searches |

The protocol is part of an SRV name. This list describes the inspected TCP examples, not every transport or port a service can use.

Northstar's client-side LDAP discovery query was:

```powershell
Resolve-DnsName _ldap._tcp.dc._msdcs.ad.northstarsolutions.com -Type SRV
```

Inspect AD-created records before making changes. Do not replace them with manually invented records merely to reproduce a lesson.

---

## Forward and Reverse DNS

Forward lookup finds an address from a hostname. Reverse lookup uses its own DNS records; it does not search A records backwards.

For Northstar's IPv4 subnet:

| Item | Value |
|---|---|
| Subnet | `192.168.252.0/24` |
| Reverse Zone | `252.168.192.in-addr.arpa` |
| Host Portion | `10` |
| Full PTR Owner | `10.252.168.192.in-addr.arpa` |
| PTR Target | `ns-dc01.ad.northstarsolutions.com` |

The corresponding A and PTR records were validated in both directions. An A record and a PTR record are distinct objects; do not assume that creating or changing one has automatically updated the other.

---

## Secure Dynamic DNS Updates

Dynamic updates allow clients or other authorized services to register and maintain DNS records. For Active Directory-integrated zones, secure updates use authentication and permissions to control those changes. Authenticated access does not imply permission to modify every record. See [Microsoft's dynamic DNS update documentation](https://learn.microsoft.com/en-us/windows-server/networking/dns/dynamic-update).

The Northstar notes record an AD-integrated reverse zone configured for secure updates only. The retained DNS screenshot demonstrates lookup results, not the zone's update-policy setting.

---

## DNS-Only Validation and Result Interpretation

Windows name resolution can involve more than one source or protocol. A useful test specifies the name, record type, and intended DNS server instead of relying only on default resolver behavior.

On `NS-DC01`, the recorded tests were:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Type A -Server 127.0.0.1 -DnsOnly
Resolve-DnsName 192.168.252.10 -Type PTR -Server 127.0.0.1 -DnsOnly
```

The server argument points to the DNS service running locally on the DC. From `NS-W11-01`, an explicitly targeted query would use the DC's network address, `192.168.252.10`, instead of loopback.

`-DnsOnly` selects the DNS protocol and excludes LLMNR/NetBIOS queries. `-NoHostsFile` is a separate option to skip the hosts file. These switches should not be described as universal cache-bypass options. See the [Resolve-DnsName reference](https://learn.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname?view=windowsserver2025-ps).

### Northstar Observation

The explicit tests displayed the expected A and PTR records in `Section = Answer`. Ordinary queries in the same screenshot displayed `Section = Question`.

The observation justified comparing query methods. It did not establish the source of the ordinary results, and the Answer section alone does not prove an authoritative response. DNS represents authoritative-answer status with a separate flag. See the [DNS message specification](https://www.rfc-editor.org/rfc/rfc1035#section-4.1.1).

The supported lab conclusion is that the explicit DNS queries returned the expected records. This was a validation observation rather than a formal incident.

### DNS Name Case

For the Northstar names used here, capitalization does not identify a different DNS host:

```text
NS-DC01.ad.northstarsolutions.com
ns-dc01.ad.northstarsolutions.com
```

The capitalization difference in the retained output was not treated as a separate DNS problem.

---

## Organizational Units and Security Groups

An Organizational Unit provides administrative organization, delegation scope, and a location to which Group Policy can be linked. A security group represents membership used in authorization. These are different controls.

| Object or Property | Role |
|---|---|
| User account | Identity |
| OU placement | Administrative location and potential policy scope |
| Security-group membership | Membership used for access assignments |
| Department / title / manager | Directory attributes; not automatic group assignments |

Northstar created Users, Workstations, Groups, Service Accounts, and Disabled Objects OUs, with departmental branches beneath Users and Workstations. The DC remains in the default Domain Controllers OU.

Moving a user to another OU does not by itself change group membership. Updating Department does not automatically move the user. Noah Wilson's transfer exercise required separate changes to attributes, manager, OU, and departmental group.

The OU structure prepares for Group Policy. It does not prove that a new policy has already been created or applied.

---

## Active Directory User Identifiers

| Identifier | Meaning | Example |
|---|---|---|
| sAMAccountName | Account logon name used with the NetBIOS domain | `NORTHSTAR\ava.chen` |
| User Principal Name (UPN) | Logon name with a UPN suffix | `ava.chen@ad.northstarsolutions.com` |
| Distinguished Name (DN) | Object name and location in the directory hierarchy | See below |

Example DN from the user-placement validation:

```text
CN=Ava Chen,OU=Finance,OU=Users,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
```

A UPN resembles an email address but does not establish that a mailbox exists. Moving an account changes its directory location; it does not create a different person or automatically grant new business access.

Northstar verified account names, enabled state, title, department, and DN with `Get-ADUser`. The manager relationships are documented in the lab notes rather than the selected screenshot.

---

## Active Directory Group Types and Scopes

**Security** groups can be used to assign permissions. **Distribution** groups are intended for distribution purposes and cannot be used to assign permissions in the same way. Scope controls allowed memberships and where a group can be used.

| Scope | Common Design Use |
|---|---|
| Global | Group accounts from the same domain by role or department |
| Domain Local | Assign permissions to resources in the group's domain |
| Universal | Combine memberships across domains within a forest where needed |

These are design uses, not a complete nesting-rules table. Consult [Microsoft's Active Directory security-group reference](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups) when designing membership across domains or changing scope.

Northstar uses `GG-Finance-Users`, `GG-IT-Users`, and `GG-Operations-Users` as Global Security groups. Distribution groups and Universal groups were not created merely for demonstration.

### AGDLP

```text
Accounts → Global Groups → Domain Local Groups → Permissions
```

Northstar has implemented the first relationship:

```text
Ava Chen + Daniel Kim → GG-Finance-Users
```

Resource-oriented Domain Local groups and permissions will be added when file services or other actual resources exist. Departmental group creation alone does not establish resource access.

### Membership Verification

On `NS-DC01`, a recorded check was:

```powershell
Get-ADGroupMember 'GG-Finance-Users' | Select-Object Name,SamAccountName
```

Validation exposed incorrect initial departmental assignments. The assignments were corrected and rechecked. Record this as a configuration-validation finding rather than inventing a deliberately induced incident.

---

## Identity Lifecycle Administration

Identity administration extends beyond creating an account:

```text
Onboard → Assign role and access → Transfer / change role → Offboard
```

Northstar used the fictional account `noah.wilson`:

| Stage | Administrative Work |
|---|---|
| Onboarding | Finance identity, Finance OU, attributes, manager, and departmental group |
| Transfer | Operations title, department, manager, OU, and group; old departmental membership removed |
| Offboarding | Account disabled, departmental memberships removed, object moved to Disabled Objects and retained |

The final output showed `Enabled = False` and `Domain Users` as the remaining displayed membership. The exercise demonstrated directory administration, not a complete organization-wide offboarding process.

Real offboarding also requires reviewing existing sessions, application access, devices, and other systems according to organizational policy. Do not assume that a disabled AD account proves every existing session or external entitlement has been revoked.

### Disabled, Locked, and Deleted

| State | Meaning |
|---|---|
| Disabled | Administrative restriction on using the retained account for authentication |
| Locked | Authentication failures have triggered an account-lockout policy |
| Deleted | The object has been removed from the active directory |

Noah's exercise used disabling, not an account lockout or deletion. Disabling and moving an account are separate actions; neither should be described as automatically clearing group membership.

---

## Domain Join and the DNS Prerequisite

Joining a Windows computer to Active Directory establishes its domain computer identity and trust relationship. It does not delete the computer's local user accounts.

Northstar's recorded transition was:

| Check | Before | After |
|---|---|---|
| Domain | `WORKGROUP` | `ad.northstarsolutions.com` |
| Part of Domain | `False` | `True` |
| IPv4 Address Assignment | VMware DHCP | VMware DHCP retained |
| IPv4 DNS Server | `192.168.252.2` | `192.168.252.10` |

Before the DNS change, the client could reach the DC by IP but could not resolve the domain or LDAP SRV records. After it was configured to use internal DNS, those lookups succeeded according to the lab notes.

The lesson is to test addressing, name resolution, and service discovery separately. A successful ping alone is not a complete domain-readiness check. This was pre-join validation, not a manufactured domain-join incident.

---

## Domain Controller Discovery and Secure Channel

On `NS-W11-01`, DC discovery was checked using:

```cmd
nltest /dsgetdc:ad.northstarsolutions.com
```

It located `NS-DC01.ad.northstarsolutions.com` at `192.168.252.10`. Discovery identifies a DC; a separate check verified the workstation's secure channel.

In an administrative PowerShell session on the workstation:

```powershell
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
Test-ComputerSecureChannel -Verbose
```

The first command showed domain membership, and the second returned `True` with a healthy-channel message. The secure-channel test is for a domain member; it should not be used as the equivalent DC health check on `NS-DC01`. See [Microsoft's Test-ComputerSecureChannel reference](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/test-computersecurechannel?view=powershell-5.1).

A failed check requires investigation of the actual error, execution context, connectivity, and trust state. Do not automatically reset trust or rejoin the domain as the first action.

---

## Active Directory Computer Objects

The domain workstation is represented by an enabled computer object at:

```text
CN=NS-W11-01,OU=Finance,OU=Workstations,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
```

Northstar performed domain joining and final OU placement as separate administrative actions. The resulting location prepares the Finance workstation for future GPO scope.

Server-side verification used:

```powershell
Get-ADComputer NS-W11-01 -Properties DistinguishedName,Enabled | Select-Object Name,Enabled,DistinguishedName
```

The object being enabled and in the correct OU does not by itself prove user login, a healthy secure channel, or successful policy application. Each capability has its own validation.

---

## Local Accounts, Domain Accounts, and User Tokens

| Identity | Account Location |
|---|---|
| `NS-W11-01\labadmin` | Local workstation |
| `NORTHSTAR\ava.chen` | Northstar Active Directory |

Both identity types can exist on the same domain-joined workstation. Check the actual account context rather than relying only on the displayed name.

Northstar inspected Ava's session with:

```powershell
whoami
whoami /user
whoami /groups
$env:USERDOMAIN
$env:USERDNSDOMAIN
```

The results showed the domain identity and `NORTHSTAR\GG-Finance-Users` in the Windows security token. Directory group membership and the groups represented in an existing session are related but distinct observations; membership changes may require signing out and back in to obtain a new logon token.

### Least Privilege

Department or job title does not establish a need for administrator rights. Northstar did not make its ordinary IT employee accounts Domain Admins solely because they work in IT.

For Ava's workstation session, the displayed token contained no `BUILTIN\Administrators` entry and the local Administrators listing did not individually include Ava. Together these support the recorded standard-user session. A direct user-membership check alone is insufficient because privileges can also come through groups. See [Microsoft's least-privilege administrative guidance](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models).

These observations are not an audit of every resource permission. The secure-channel success is documented in a separate administrative session, not inferred from the unfinished command visible at the bottom of Ava's screenshot.

---

## Phase 4–5 Learning Checkpoint

Completed work now includes DNS records, reverse resolution, OUs, domain identities, departmental groups, an identity lifecycle, and a domain-joined workstation. No intentionally induced Phase 4–5 fault scenario is reported complete.

Phase 6 — Centralized Group Policy is next and has not yet started hands-on. Domain Local resource groups, file-service permissions, and automation remain future capabilities. See the [server module overview](../02-windows-server/README.md) for the recorded project status.

---

## Production vs Homelab

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
