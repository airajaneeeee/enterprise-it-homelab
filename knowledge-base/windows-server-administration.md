# Windows Server Administration — Knowledge Base

This knowledge base explains concepts practiced through Phase 7, including centralized Group Policy and Windows DHCP. Lab examples refer to the recorded Northstar environment; planned capabilities are identified separately.

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

The original VMware DHCP allocation range was:

`192.168.252.128-192.168.252.254`

During Phase 7, VMware DHCP was disabled and Windows DHCP on `NS-DC01` took over client address allocation. VMware still provides NAT through `192.168.252.2`.

### Lease Process

The initial IPv4 lease exchange is commonly summarized as DORA:

```text
Discover → Offer → Request → Acknowledge
```

The client requests configuration and the DHCP server grants a lease. This describes the initial exchange, not every renewal. Northstar verified leases and client configuration; it did not retain a packet capture demonstrating each DORA message. See [Microsoft's DHCP training overview](https://learn.microsoft.com/en-gb/training/modules/deploy-manage-dynamic-host-configuration-protocol/).

### Scope, Exclusion, Reservation, and Static Address

| Term | Meaning | Northstar Example |
|---|---|---|
| Scope | Address range and associated configuration managed for a subnet | `192.168.252.20`–`192.168.252.199` |
| Exclusion | Addresses within the scope withheld from allocation | `.20`–`.49`, planned infrastructure space |
| Lease | A client address assignment with a duration | Initial workstation lease `.50`, eight-day scope duration |
| Reservation | A predictable DHCP assignment associated with a client identifier | `.120` for MAC `00-0C-29-5F-31-87` |
| Manual static address | Address configured directly on the system | `NS-DC01` at `.10` |

A reservation still requires the client to obtain its address through DHCP. An exclusion does not configure an address on a server or create a reservation. See [Microsoft's DHCP scope documentation](https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/dhcp-scopes).

Northstar's client range is `.50`–`.199`, including the `.120` reservation. That reserved address is assigned to the Finance workstation rather than available for unrelated dynamic clients.

### Scope Options and DNS

| Option | Purpose | Northstar Value |
|---|---|---|
| 003 — Router | Client default gateway | `192.168.252.2` |
| 006 — DNS Servers | DNS servers the client should query | `192.168.252.10` |
| 015 — DNS Domain Name | DNS domain suffix supplied to the client | `ad.northstarsolutions.com` |

These values were configured at scope level. DHCP options can also be configured at other levels; inspect the client's actual configuration as well as the intended scope values. See [Microsoft's DHCP configuration guide](https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/quickstart-install-configure-dhcp-server).

Option 006 configures the client's DNS server address. It does not configure a DNS forwarder on `NS-DC01`. Option 015 also does not join a workstation to Active Directory.

Northstar changed the workstation's DNS configuration from manual to automatic during cutover so validation tested the DHCP-delivered DNS setting.

### Authorization, Scope Activation, and Cutover

Installing the role, authorizing a Windows DHCP server in Active Directory, and activating a scope are separate steps. Microsoft's [DHCP deployment guide](https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/quickstart-install-configure-dhcp-server) describes these configuration tasks.

Northstar's recorded sequence was:

1. Install DHCP and complete post-installation configuration on `NS-DC01`.
2. Authorize the server and verify it with `Get-DhcpServerInDC`.
3. Configure the scope, exclusion, and options while leaving the scope inactive.
4. Disable VMware DHCP on VMnet8 while retaining NAT.
5. Activate the Windows scope.
6. Renew the client configuration and check both the lease and delivered options.

Keeping the new scope inactive until VMware DHCP was disabled prevented the two independent configurations from competing during this lab cutover. AD authorization was not used as a substitute for disabling VMware DHCP.

### Administration and Client Validation

On `NS-DC01`, inspect the server and scope with:

```powershell
Get-DhcpServerInDC
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 192.168.252.0
Get-DhcpServerv4Reservation -ScopeId 192.168.252.0
Get-DhcpServerv4OptionValue -ScopeId 192.168.252.0
```

On the client, `ipconfig /all` identifies the lease address, DHCP server, gateway, DNS server, and suffix. During the controlled cutover and reservation tests, Northstar also used:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

Lease release changes connectivity, so use it as a deliberate configuration step rather than a prerequisite for every inspection.

The workstation first received `.50` from Windows DHCP, then `.120` after its reservation was created and the lease renewed. `DHCP Enabled = Yes` remained visible. These are successive observations, not conflicting static-address assignments.

---

## Static Addressing

Core infrastructure systems commonly require predictable addressing.

`NS-DC01` was configured with:

`192.168.252.10`

This address was outside the original VMware DHCP allocation range and is also outside the current Windows DHCP scope.

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

The OU structure supports Group Policy targeting. Phase 6 separately verified applied computer and user policies; the existence of the OU hierarchy alone does not prove policy application.

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

Northstar's recorded Phase 5 transition was:

| Check | Before | After |
|---|---|---|
| Domain | `WORKGROUP` | `ad.northstarsolutions.com` |
| Part of Domain | `False` | `True` |
| IPv4 Address Assignment | VMware DHCP | VMware DHCP retained |
| IPv4 DNS Server | `192.168.252.2` | `192.168.252.10` |

Before the DNS change, the client could reach the DC by IP but could not resolve the domain or LDAP SRV records. After it was configured to use internal DNS, those lookups succeeded according to the lab notes.

The lesson is to test addressing, name resolution, and service discovery separately. A successful ping alone is not a complete domain-readiness check. This was pre-join validation, not a manufactured domain-join incident.

In Phase 7, Windows DHCP replaced VMware DHCP. The final workstation address is the DHCP reservation `192.168.252.120`, and internal DNS at `192.168.252.10` is now supplied by Option 006. The table above preserves the earlier domain-join checkpoint.

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

Northstar performed domain joining and final OU placement as separate administrative actions. The resulting location supports Finance workstation policy targeting, which was validated separately in Phase 6.

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

## Group Policy Scope and Validation

A Group Policy Object contains settings, while its links help determine where those settings apply. GPOs can be linked to sites, domains, and OUs. A GPO existing in the domain does not by itself establish that it applies to a particular user or computer. See [Microsoft's Group Policy overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview).

### Computer and User Context

Northstar validated two separate contexts:

| Policy | Context | Recorded Result |
|---|---|---|
| `Northstar - Workstation Baseline` | Computer `NS-W11-01` | Applied; `InactivityTimeoutSecs = 900` |
| `Northstar - Finance User Policy` | User `NORTHSTAR\ava.chen` | Applied in the Finance session |

Computer-scope checks were run in an elevated workstation session:

```powershell
gpresult /r /scope computer
gpresult /h C:\gpo-report.html /scope computer
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System' -Name InactivityTimeoutSecs
```

User-scope checks were run in Ava's session:

```cmd
whoami
gpresult /r /scope user
```

Check identity before interpreting user results. Running a user-scope query in a different account's session does not demonstrate Ava's applied policies. `gpresult` reports resultant policy information; see [Microsoft's Group Policy Results documentation](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-modeling-results).

The workstation screenshot verifies the applied GPO and a 900-second registry value. It does not show a timed lock test or the exact GPO link location. The Finance evidence establishes policy application without identifying the exact restriction setting.

### Refresh vs Diagnosis

`gpupdate /force` was used after correcting the Finance policy link. A successful refresh should be followed by applied-policy and functional checks. It does not by itself prove that the intended GPO is in scope. See [Microsoft's policy-application troubleshooting guidance](https://learn.microsoft.com/en-ca/troubleshoot/windows-server/group-policy/applying-group-policy-troubleshooting-guidance).

Northstar's **SIM-GPO-001** followed this sequence:

1. Identify the affected Finance user and missing policy behavior.
2. Verify that the user remains in the intended OU.
3. Confirm that the GPO exists and its relevant setting remains enabled.
4. Inspect `gpresult` and observe the absent applied GPO.
5. Review scope and identify the missing Finance user OU link.
6. Restore the link and refresh policy.
7. Confirm the GPO in user-scope results and retest the restriction.

The missing link was the recorded cause in this controlled scenario. The screenshot shows policy absence and recovery; link restoration and the functional retest are recorded in the notes. This example is not a complete diagnostic checklist for every Group Policy failure.

---

## Troubleshooting DHCP-Delivered DNS

A valid lease establishes address allocation. It does not establish correct internal name resolution, domain discovery, or a healthy secure channel.

In **SIM-DHCP-001**, the client retained its `.120` reservation while Option 006 supplied `1.1.1.1`. External DNS worked, but Northstar names failed. A direct query to the internal DNS server succeeded.

### Compare Configured and Explicit Resolvers

Inspect client configuration first:

```cmd
ipconfig /all
```

Then compare the normal lookup path with a query explicitly targeting the internal DNS server:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Server 192.168.252.10 -DnsOnly
Resolve-DnsName _ldap._tcp.dc._msdcs.ad.northstarsolutions.com -Type SRV -Server 192.168.252.10 -DnsOnly
Resolve-DnsName microsoft.com
```

The successful direct query, combined with the wrong DNS client setting, isolated the recorded fault to the DHCP-delivered resolver address. It did not indicate that the internal DNS service needed rebuilding.

Northstar restored Option 006 to `.10`, renewed the client's configuration, cleared the DNS cache with `ipconfig /flushdns`, and repeated internal resolution, DC discovery, and secure-channel checks. Cache clearing alone would not correct an incorrect DHCP option.

### Interpret the Actual DNS Answer

When testing service discovery, request `-Type SRV` and inspect the returned record type. Screenshot 24's LDAP service-name query omitted that switch and displayed an SOA record in the Authority section. That output is not an SRV answer. The lab notes separately record explicit SRV validation.

An earlier attempt to reproduce the fault using VMware's `.2` resolver still resolved internal names and SRV queries according to the notes. Preserve that result without inventing a caching or forwarding explanation. A test that does not reproduce the expected failure is still useful evidence.

---

## Windows Time and Secure-Channel Investigation

Time inspection became part of Northstar's troubleshooting when `Test-ComputerSecureChannel` returned `False` despite successful DNS and DC discovery checks.

### Inspect Before Changing Configuration

On the domain workstation, inspect the clock and configured time source:

```powershell
Get-Date
Get-TimeZone
w32tm /query /source
w32tm /query /status
w32tm /stripchart /computer:ns-dc01.ad.northstarsolutions.com /samples:5 /dataonly
```

`/query /source` identifies the selected time source; `/stripchart` measures offsets against the specified target. These answer different questions. After correcting the workstation clock, Northstar used `w32tm /resync` to request synchronization. See [Microsoft's Windows Time tools reference](https://learn.microsoft.com/en-au/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings).

### Preserve Conflicting Results

During the unexpected lab issue:

| Observation | Recorded Result |
|---|---|
| `Test-ComputerSecureChannel -Verbose` | Initially `False` |
| Domain membership | Still `True` |
| `nltest /sc_verify` and `/sc_query` | `NERR_Success` |
| DNS and DC discovery | Working |
| Workstation clock | Incorrect time identified in the notes |
| After clock correction and synchronization | Secure-channel test returned `True` |

Recovery evidence showed `NS-DC01` as the time source and offsets around 1.7–1.9 milliseconds during the retained samples. The workstation's time-zone ID was `Pacific Standard Time`; this does not establish the server's exact time-zone configuration.

The lesson is to record each result and investigate contributing conditions before making disruptive trust changes. Recovery followed time correction, but the evidence does not prove DHCP caused the issue or establish time as its sole cause. This was an unexpected project incident, distinct from the two controlled simulations.

### Run Tools Where They Are Available

`Get-ADComputer` was not recognized on the workstation because its Active Directory PowerShell tools were unavailable. The query was run on `NS-DC01`, where the tools were installed. A missing local command should be distinguished from a directory service failure.

---

## Phase 7 Learning Checkpoint

Completed work includes DNS records, reverse resolution, OUs, domain identities, departmental groups, an identity lifecycle, domain-client integration, computer and user Group Policy, and Windows DHCP administration.

Phase 6–7 troubleshooting includes the controlled simulations `SIM-GPO-001` and `SIM-DHCP-001`, plus the unexpected workstation time and secure-channel issue. The earlier membership correction and pre-join DNS observation remain validation findings.

Phase 8 — File Services and Permissions is next. Domain Local resource groups, file-service permissions, and automation remain future capabilities. See the [server module overview](../02-windows-server/README.md) for the recorded project status.

---

## Production vs Homelab

The Northstar environment practices production-oriented concepts but is not presented as a production deployment.

Current homelab limitations include:

- Single physical host
- Single Domain Controller
- Single DNS Server
- DHCP hosted on the same DC, with no failover partner
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
