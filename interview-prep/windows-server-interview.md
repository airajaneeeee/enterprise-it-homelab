# Windows Server Administration — Interview Preparation

This document contains interview questions and sample answers based on the Windows Server, DNS, directory administration, workstation integration, Group Policy, and DHCP work recorded through Phase 7 of the Northstar Solutions enterprise IT homelab.

Answers distinguish completed lab actions from hypothetical troubleshooting approaches and future work. Technical detail and supporting evidence are maintained in the [server baseline](../02-windows-server/server-baseline.md), [screenshot guide](../02-windows-server/screenshots/README.md), and [server knowledge base](../knowledge-base/windows-server-administration.md).

---

## 1. Windows Server and Network Configuration

### 1. Why would you assign a static IP address to a server?

A server providing core infrastructure services should generally have predictable network addressing so clients and other systems can reliably locate its services.

In my Northstar homelab, `NS-DC01` originally received `192.168.252.129` through VMware DHCP.

Before deploying Active Directory and DNS, I inspected the VMware subnet and DHCP allocation range and configured the server with:

`192.168.252.10`

The address was outside the original VMware DHCP allocation range and is also outside the Windows DHCP scope introduced in Phase 7.

In production, I would follow the organization's IP address management and network design standards rather than independently selecting an address.

### 2. What is the difference between DHCP and static addressing?

DHCP automatically provides network configuration such as an IP address, subnet mask, gateway, and DNS server information.

A manually configured static address remains predictable unless an administrator changes it.

In my lab, I changed `NS-DC01` from DHCP to static addressing before making it a Domain Controller and DNS Server.

### 3. What does a default gateway do?

A default gateway provides a path toward destinations outside the host's local subnet.

`NS-DC01` is connected to:

`192.168.252.0/24`

and uses the VMware NAT gateway:

`192.168.252.2`

I verified gateway connectivity after configuring the server's static address.

### 4. How would you distinguish a DNS problem from general network connectivity?

I would test the functions separately.

For example, I can verify the system's IP configuration, test the local gateway, and then test an external IP address.

If IP connectivity works but hostname resolution fails, I would investigate DNS rather than assuming that general network connectivity is unavailable.

I observed this directly in my lab. Before DNS Server was installed, `NS-DC01` successfully reached `8.8.8.8`, but a DNS query to its configured DNS address failed because the server was pointing to itself before the DNS service existed.

After DNS Server deployment, external DNS resolution succeeded.

---

## 2. Windows Server Roles and Active Directory

### 1. What is the difference between a Windows Server role and a feature?

A server role represents a primary responsibility performed by the server, such as Active Directory Domain Services, DNS Server, or DHCP Server.

A feature provides additional operating-system functionality or supports installed roles.

In my lab, I installed the Active Directory Domain Services and DNS Server roles on `NS-DC01`, then added DHCP Server in Phase 7.

### 2. What is Active Directory Domain Services?

Active Directory Domain Services provides centralized directory, identity, authentication, and management capabilities for Windows domain environments.

It can centrally manage objects such as users, computers, groups, and Organizational Units and supports technologies such as Group Policy.

In my homelab, I installed AD DS on Windows Server 2025 and promoted `NS-DC01` as the first Domain Controller.

### 3. What is the difference between a forest and a domain?

A forest is the top-level logical structure of an Active Directory environment and can contain one or more domains.

A domain is a logical administrative and security structure containing directory objects such as users, computers, and groups.

My Northstar environment currently uses a single-domain forest:

`ad.northstarsolutions.com`

### 4. What is a Domain Controller?

A Domain Controller is a Windows Server running Active Directory Domain Services and providing directory services for an Active Directory domain.

Domain Controllers participate in authentication, directory queries, Group Policy processing, and Active Directory service discovery.

My first Northstar Domain Controller is:

`NS-DC01.ad.northstarsolutions.com`

### 5. Does installing the AD DS role automatically make a server a Domain Controller?

No.

Installing the Active Directory Domain Services role installs the required server components, but the server must still be promoted to a Domain Controller.

In my lab, I first installed AD DS and DNS Server and then promoted `NS-DC01` as the first Domain Controller in a new forest.

### 6. What happens when you promote the first Domain Controller in a new forest?

In my lab, promotion created the new Active Directory forest and root domain:

`ad.northstarsolutions.com`

It configured the NetBIOS domain name:

`NORTHSTAR`

and configured `NS-DC01` as a Domain Controller, DNS Server, and Global Catalog.

After promotion, the server restarted and I validated the resulting domain and forest configuration.

---

## 3. Active Directory and DNS

### 1. Why is DNS important to Active Directory?

Active Directory depends heavily on DNS.

Domain clients use DNS not only for normal hostname resolution but also to locate Domain Controllers and Active Directory services.

Because of that, incorrect DNS configuration can cause domain joins, authentication, Group Policy, and other Active Directory operations to fail.

### 2. Why does your Domain Controller use itself as its DNS server?

`NS-DC01` hosts DNS for the Northstar Active Directory environment, so it uses its own local DNS service.

Its current IPv4 DNS client setting is:

`127.0.0.1`

which is the IPv4 loopback address for the same server.

`NS-DC01` itself is reachable on the network at:

`192.168.252.10`

This allows the Domain Controller to use the Active Directory-aware internal DNS infrastructure.

My domain-joined workstation, `NS-W11-01`, uses that internal DNS service through the server's network address, `192.168.252.10`. The loopback address is used on the DC itself, not as the workstation's DNS server.

### 3. Why shouldn't an Active Directory client simply use a public DNS server such as 8.8.8.8?

Public DNS resolvers do not contain the private Active Directory DNS records required for the Northstar domain.

Domain clients need to query the internal DNS infrastructure so they can locate Domain Controllers and other Active Directory services.

The internal DNS server can handle the Active Directory namespace while still providing resolution for external names.

### 4. How did you verify DNS after deployment?

I tested both internal and external DNS resolution.

For the internal environment, I used:

```powershell
Resolve-DnsName NS-DC01.ad.northstarsolutions.com
```

and confirmed that the Domain Controller FQDN resolved to:

`192.168.252.10`

I then tested:

```powershell
Resolve-DnsName microsoft.com
```

and confirmed that external DNS resolution also succeeded.

This allowed me to verify internal and external resolution separately.

### 5. What was the DNS delegation warning you encountered during Domain Controller promotion?

During creation of the new forest, the wizard indicated that a DNS delegation could not be created because an authoritative parent zone could not be found.

In my isolated homelab, there was no existing authoritative parent DNS infrastructure managing the simulated namespace.

I therefore did not create a delegation, and the warning did not prevent successful deployment of the new forest and DNS environment.

### 6. Do you currently use DNS forwarders?

No explicit DNS forwarders are currently configured on `NS-DC01`.

External DNS resolution still succeeds because Windows DNS can use root hints when no forwarder is configured.

This helped me distinguish between the DNS client setting on the server and the forwarding behavior of the DNS Server service.

### 7. What is the difference between a DNS client setting and a DNS forwarder?

A DNS client setting tells a computer which DNS server it should query.

A DNS forwarder tells a DNS server where it should send queries that it cannot answer from its own authoritative zones or cache.

In my lab, `NS-DC01` uses its own local DNS service.

No explicit forwarder is currently configured, so external lookups can be resolved using root hints.

### 8. What does `::1` mean when `nslookup` shows it as the DNS server?

`::1` is the IPv6 loopback address.

It refers back to the local computer, similar to `127.0.0.1` in IPv4.

When `nslookup` displayed `::1`, it was still attempting to query the DNS service running locally on `NS-DC01`.

### 9. Why didn't you disable IPv6 when `nslookup` showed `::1`?

I did not treat the use of the IPv6 loopback address as a reason to disable IPv6.

I kept IPv6 enabled and verified the environment separately.

The server showed IPv4 Internet connectivity and no active IPv6 traffic during the baseline check, while internal and external DNS resolution were both successfully verified.

---

## 4. Active Directory Validation

### 1. How did you verify that the Domain Controller was working after promotion?

I did not rely only on the installation wizard.

After the restart, I verified the server identity and administrative context using:

```powershell
hostname
whoami
```

I then inspected the domain and forest using:

```powershell
Get-ADDomain
Get-ADForest
```

and checked the core services using:

```powershell
Get-Service NTDS,DNS
```

The results confirmed the correct domain and forest, the Domain Controller, Global Catalog configuration, and that both Active Directory Domain Services and DNS Server were running.

I also tested internal and external DNS resolution.

### 2. What is a Global Catalog?

A Global Catalog is a Domain Controller capability that provides a searchable representation of objects across an Active Directory forest and supports important directory operations.

During my lab, `NS-DC01` was configured as a Global Catalog.

I verified that configuration using `Get-ADForest`.

### 3. What is DSRM?

DSRM stands for Directory Services Restore Mode.

It is a special recovery mode used for Active Directory maintenance and recovery.

I configured a DSRM password during Domain Controller promotion, but I do not store that password in my public project documentation.

### 4. What are forest and domain functional levels?

Forest and domain functional levels determine which Active Directory capabilities are available and which Windows Server versions can participate as Domain Controllers.

My lab uses Windows Server 2025 forest and domain functional levels because the environment is being built using Windows Server 2025 Domain Controller infrastructure.

In production, compatibility with existing infrastructure would need to be evaluated before changing functional levels.

### 5. What are FSMO roles?

FSMO stands for Flexible Single Master Operations.

Active Directory uses several FSMO roles for directory operations that require a single authoritative role holder.

During validation of my new single-Domain-Controller forest, `NS-DC01` appeared as the role holder for the domain and forest roles returned by `Get-ADDomain` and `Get-ADForest`.

The roles included:

- PDC Emulator
- RID Master
- Infrastructure Master
- Schema Master
- Domain Naming Master

I will be studying FSMO role administration in more depth later in the project.

### 6. What server-management settings did you verify before continuing with Active Directory administration?

Before moving into user, group, and OU administration, I completed a short post-deployment server baseline.

I verified that:

- The network profile was `DomainAuthenticated`.
- Remote Desktop was enabled.
- Network Level Authentication remained enabled.
- Remote Management was enabled.
- Windows Defender Firewall remained enabled.
- An ICMPv4 Echo Request inbound rule was enabled for controlled connectivity testing.
- IPv6 remained enabled.
- VMware Tools was operational.
- The time zone was correct.
- Windows Update settings/status were checked.

The goal was to prepare the server for realistic administration and troubleshooting without weakening the server by disabling the firewall or unnecessarily disabling IPv6.

---

## 5. DNS Record Administration

### 1. What is the difference between an A record and a PTR record?

An A record maps a hostname to an IPv4 address. A PTR record maps a reverse DNS name to a hostname.

In my lab, I verified:

```text
Forward: ns-dc01.ad.northstarsolutions.com → 192.168.252.10
Reverse: 192.168.252.10 → ns-dc01.ad.northstarsolutions.com
```

I configured the reverse lookup zone for the `192.168.252.0/24` network and validated the A and PTR results using DNS-only queries directed at the local DNS service on `NS-DC01`.

### 2. Does DNS search A records backwards to perform a reverse lookup?

No. Reverse DNS uses its own zone and PTR records.

For my lab subnet, the reverse zone is `252.168.192.in-addr.arpa`. The PTR owner for host `.10` is `10.252.168.192.in-addr.arpa`, and it points to the DC hostname.

I tested forward and reverse resolution separately rather than assuming one successful lookup proved both records were correct.

### 3. What is a CNAME record, and how did you practice using one?

A CNAME is an alias that points to another DNS name rather than directly to an IP address.

My temporary exercise used:

```text
dns-alias.ad.northstarsolutions.com
                 ↓
dns-lab.ad.northstarsolutions.com
                 ↓
192.168.252.10
```

After validation, I removed both training records. They were an administration exercise, not additional permanent servers. A successful alias lookup demonstrated name resolution, not the availability of a new application service.

### 4. What is an SRV record, and why does Active Directory use it?

An SRV record identifies a service's target host and port, with priority and weight information. Active Directory clients use these records to discover services such as LDAP, Kerberos, and the Global Catalog.

I inspected AD-created records and checked LDAP SRV resolution from the workstation before joining it to the domain:

```powershell
Resolve-DnsName _ldap._tcp.dc._msdcs.ad.northstarsolutions.com -Type SRV
```

That helped me connect DNS configuration with Domain Controller discovery rather than treating DNS only as hostname-to-address mapping.

### 5. Why did you select secure dynamic updates for the reverse zone?

The zone is Active Directory-integrated, so I configured secure updates to use authentication and permissions for record changes rather than allowing arbitrary unauthenticated updates.

The setting is recorded in my lab notes. My retained DNS screenshot shows the lookup results rather than the zone's update-policy properties.

In a production environment, I would assess the clients and services that need to register records before selecting or changing the update policy.

### 6. What did you learn from comparing default and DNS-only queries?

The ordinary queries in my screenshot displayed `Section = Question`, while the explicit queries returned the expected records in `Section = Answer`.

For the explicit PTR test on `NS-DC01`, I used:

```powershell
Resolve-DnsName 192.168.252.10 -Type PTR -Server 127.0.0.1 -DnsOnly
```

I learned to specify the server, record type, and query method when validating DNS. I would not infer the source of the ordinary results from that display alone or treat the Answer section as proof of the authoritative-answer flag.

This was a validation observation, not a formal troubleshooting incident.

---

## 6. Active Directory Administration

### 1. What is an Organizational Unit, and how did you use OUs?

An OU is an Active Directory container used for administrative organization, delegation, and Group Policy targeting.

I created a top-level `Northstar` OU with Users, Workstations, Groups, Service Accounts, and Disabled Objects branches. Users and Workstations have Finance, IT, and Operations sub-OUs.

I kept `NS-DC01` in the default Domain Controllers OU. The custom structure supports administration and policy targeting. In Phase 6, I separately validated computer and user policy application; OU creation alone does not demonstrate that policies apply.

### 2. What is the difference between an OU and a security group?

An OU provides an administrative location and potential policy scope. A security group represents membership that can be used in authorization.

Moving a user to another OU does not automatically change group membership. During Noah Wilson's fictional transfer from Finance to Operations, I separately changed his OU, department and title, manager, and departmental group membership.

That demonstrated why changing a directory attribute alone is not a complete access-transfer process.

### 3. What is a Distinguished Name?

A Distinguished Name identifies an object and its location within the directory hierarchy.

For example, Ava Chen's recorded DN is:

```text
CN=Ava Chen,OU=Finance,OU=Users,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
```

I used `Get-ADUser` to verify placement after creating the employee accounts. The DN is different from the logon identities `NORTHSTAR\ava.chen` and `ava.chen@ad.northstarsolutions.com`.

### 4. How do you use Global and Domain Local groups?

A common approach is to use Global groups to collect accounts by role or department and Domain Local groups to assign access to resources in the resource domain.

I created `GG-Finance-Users`, `GG-IT-Users`, and `GG-Operations-Users` as Global Security groups and verified departmental memberships.

Resource-oriented Domain Local groups are future work for the file-services phase. I have not claimed resource permissions simply because departmental groups exist.

### 5. What is AGDLP, and how much have you implemented?

AGDLP describes this access-control model:

```text
Accounts → Global Groups → Domain Local Groups → Permissions
```

I have implemented Accounts → Global Groups. Ava Chen and Daniel Kim, for example, belong to `GG-Finance-Users`.

I will introduce resource-specific Domain Local groups and permissions when actual resources such as file shares are available. The complete model is not yet implemented in my lab.

### 6. How did you practice employee onboarding, transfer, and offboarding?

I created a fictional employee, Noah Wilson, and onboarded him into Finance with a title, department, manager, OU placement, and Finance group membership.

I then simulated a transfer to Operations and updated those controls separately. For offboarding, I disabled the account, removed departmental group membership, and moved it into the Disabled Objects OU while retaining the directory object.

The final PowerShell output showed `Enabled = False` and only `Domain Users` in the displayed group memberships.

This was a controlled identity-lifecycle exercise. Real offboarding would also involve the organization's applications, sessions, devices, and other access systems; my AD exercise does not demonstrate all of those activities.

### 7. Why would you avoid making every IT employee a Domain Admin?

Administrative privilege should follow actual responsibilities, not job title alone.

I created fictional IT Support and Systems Administrator employees but did not add their ordinary accounts to Domain Admins solely because of their roles.

I would assess required tasks and delegated access, keeping routine employee activity separate from privileged administration.

### 8. Did validation uncover any configuration mistakes?

Yes. Initial departmental memberships were incorrect. I reviewed the assignments, corrected them, and re-verified each group's membership before continuing.

I document that as a configuration-validation correction. It was not an intentionally induced support incident, and the retained screenshot shows the corrected state rather than the original mistake.

---

## 7. Domain Workstation Integration

### 1. What DNS configuration did your Active Directory client need?

The workstation needed to use Northstar's internal DNS service at `192.168.252.10` to resolve the domain and service-discovery records.

Before the change, `NS-W11-01` used VMware DNS at `192.168.252.2`. I changed its IPv4 DNS server while retaining VMware DHCP addressing.

After the change, I validated the DC name, domain, LDAP SRV records, and external name resolution before joining the domain. I did not configure the client to use the DC's loopback address `127.0.0.1`.

That was the Phase 5 configuration. In Phase 7, I migrated the workstation to Windows DHCP and automatic DNS configuration. It now receives internal DNS through Option 006 and uses the reservation `192.168.252.120`.

### 2. What did you learn when you could ping the DC but could not resolve the domain?

It demonstrated that IP connectivity, DNS resolution, and Active Directory service discovery are separate checks.

The workstation could reach `192.168.252.10`, but its VMware DNS server did not resolve the Northstar domain or LDAP SRV records. Configuring the client to use internal DNS resolved that prerequisite gap.

I recorded this as a pre-join observation, not a deliberately induced domain-join failure.

### 3. How did you verify the workstation's domain join?

In an administrative PowerShell session on `NS-W11-01`, I checked:

```powershell
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
nltest /dsgetdc:ad.northstarsolutions.com
Test-ComputerSecureChannel -Verbose
```

The results showed membership in `ad.northstarsolutions.com`, `CsPartOfDomain = True`, successful DC discovery, and a healthy secure channel.

I also checked the computer object in Active Directory and tested a separate Finance domain-user session. Each check validated a different part of the integration.

### 4. What is the workstation-domain secure channel?

It is part of the computer account's trust relationship with the domain. A domain-joined workstation needs more than basic network connectivity to maintain that relationship.

My administrative check on `NS-W11-01` returned `True` and reported the channel in good condition. I used the domain-member check on the workstation, not as a substitute for Domain Controller health validation.

If the check failed, I would investigate the error and context before attempting a trust repair or domain rejoin.

### 5. Where did you place the workstation's computer account?

I moved the enabled `NS-W11-01` computer object to:

```text
Northstar
└── Workstations
    └── Finance
        └── NS-W11-01
```

I verified its Distinguished Name and enabled state with `Get-ADComputer` on the server. Domain joining and final OU placement were separate actions. In Phase 6, I separately verified the workstation's applied computer policy using `gpresult`.

### 6. What is the difference between a local account and a domain account?

A local account belongs to one computer, while a domain account is stored in Active Directory and centrally managed.

For example, `NS-W11-01\labadmin` is a local identity and `NORTHSTAR\ava.chen` is a domain identity. Joining the workstation to the domain did not remove its local accounts.

I use `whoami` to verify the actual account context rather than assuming the visible username identifies the authentication source.

### 7. How did you validate the Finance domain-user session?

I signed in as `NORTHSTAR\ava.chen` and checked the identity, user SID, group token, domain environment variables, and Domain Controller discovery.

The token included `NORTHSTAR\GG-Finance-Users`. It did not show `BUILTIN\Administrators`, and the local Administrators listing did not individually include Ava.

Together those observations support the documented standard-user session; a direct user-membership check alone would not account for privileges inherited through groups. I do not describe these checks as an audit of all resource permissions.

The successful secure-channel result came from the separate administrative session. The command visible at the bottom of the Finance screenshot has no displayed result.

### 8. What would you check if a workstation could not join the domain?

I would establish the symptom and scope, then check:

1. Client IP address, subnet, gateway, and DNS configuration.
2. Connectivity to the Domain Controller.
3. Resolution of the domain and relevant AD SRV records.
4. Domain Controller availability and relevant services.
5. Time synchronization where relevant.
6. Credentials and permissions used for the join.
7. The specific error and relevant Windows logs.

My actual pre-join DNS observation illustrates the first part of this approach. The complete list is how I would investigate a failure, not a claim that I completed every failure scenario in the lab.

---

## 8. Centralized Group Policy

### 1. What Group Policy work have you completed?

I validated `Northstar - Workstation Baseline` in the computer context on `NS-W11-01` and `Northstar - Finance User Policy` in Ava Chen's user context.

For the workstation policy, I verified `InactivityTimeoutSecs = 900` in the registry and checked that the GPO appeared in computer-scope results. For the Finance policy, I verified the signed-in identity and the applied user-policy list.

My retained Finance evidence does not identify the exact restriction setting, so I describe the policy application and troubleshooting that I can support.

### 2. How did you distinguish computer policy from user policy?

I checked them in separate contexts. In an elevated session on the workstation, I used:

```cmd
gpresult /r /scope computer
gpresult /h C:\gpo-report.html /scope computer
```

In Ava's session, I used:

```cmd
whoami
gpresult /r /scope user
```

This let me verify the intended computer and user results rather than assuming that one applied-policy list described both contexts.

### 3. Does a GPO existing in the domain mean it applies to the intended user?

No. In my simulated Finance policy incident, the GPO still existed and its configured setting remained enabled, but the Finance user OU link was missing.

The user remained in the correct OU, yet the policy was absent from `gpresult`. I learned to check scope and links as well as the GPO's settings.

### 4. Describe the Group Policy problem you troubleshot.

In **SIM-GPO-001**, I deliberately reproduced a missing Finance user OU link. This was a controlled support simulation.

I checked the user's OU, confirmed that the GPO and setting still existed, and found the policy absent from the applied results. I then identified and restored the missing link.

After `gpupdate /force`, the Finance GPO appeared in user-scope results again. My notes also record that the functional restriction returned. The screenshot supports the policy absence, refresh, and recovery; it does not show the link edit itself.

### 5. Why wasn't repeatedly running gpupdate enough?

Refreshing policy did not address the missing link. I needed to correct the scope problem first, then refresh and verify the result.

The lesson was to identify why the intended policy was absent rather than treating a successful refresh message as proof that it applied.

---

## 9. Windows DHCP Administration

### 1. What did you change when you deployed Windows DHCP?

I moved client address allocation from VMware DHCP at `192.168.252.254` to Windows DHCP on `NS-DC01` at `192.168.252.10`.

VMware continued providing NAT through `192.168.252.2`. The DC retained its static address and local DNS client setting `127.0.0.1`.

The workstation changed from VMware DHCP with manually configured internal DNS to Windows DHCP with automatically supplied DNS settings.

### 2. How did you design the scope?

I configured the `Northstar Client Network` scope for `192.168.252.0/24`:

| Setting | Value |
|---|---|
| Scope Range | `192.168.252.20`–`192.168.252.199` |
| Exclusion | `192.168.252.20`–`192.168.252.49` |
| Client Range | `.50`–`.199`, including the `.120` reservation |
| Lease Duration | 8 days |

The exclusion protected the planned infrastructure range while letting me practice scope administration. The DC's static `.10` address was outside the scope.

### 3. What is the difference between an exclusion, a reservation, and a static address?

In my lab, the exclusion withheld `.20`–`.49` from DHCP allocation. It did not assign those addresses to any server.

The reservation associated `.120` with the workstation's MAC address, `00-0C-29-5F-31-87`. The workstation continued to use DHCP.

The DC's `.10` address was configured manually on the server. That is different from the client's reservation.

### 4. Which DHCP options did you configure?

| Option | Value |
|---|---|
| 003 — Router | `192.168.252.2` |
| 006 — DNS Servers | `192.168.252.10` |
| 015 — DNS Domain Name | `ad.northstarsolutions.com` |

I set the workstation to obtain DNS automatically so I could validate Option 006 rather than leave its earlier manual setting in place. I then checked the actual client configuration with `ipconfig /all`.

Option 006 supplies a DNS client setting; it does not configure a DNS forwarder on the server.

### 5. How did you manage the DHCP cutover?

I installed the role, completed post-installation configuration, and authorized `NS-DC01` in Active Directory. I verified authorization with `Get-DhcpServerInDC`.

I configured the Windows scope but kept it inactive until VMware DHCP was disabled on VMnet8. I retained NAT, activated the Windows scope, then released and renewed the client's lease.

This sequencing avoided having the two independent DHCP configurations compete during the lab migration.

### 6. How did you prove the client was using Windows DHCP?

The first renewed lease was `192.168.252.50`. Client output showed DHCP enabled, `.10` as both DHCP and DNS server, gateway `.2`, and the Northstar DNS suffix.

On the server, I used `Get-DhcpServerv4Lease -ScopeId 192.168.252.0` and matched the workstation's client ID to its active lease.

After creating the reservation and renewing again, the client received `.120` while still showing `DHCP Enabled = Yes`.

### 7. Which PowerShell commands did you use to administer DHCP?

On `NS-DC01`, I used:

```powershell
Get-DhcpServerInDC
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 192.168.252.0
Get-DhcpServerv4Reservation -ScopeId 192.168.252.0
Get-DhcpServerv4OptionValue -ScopeId 192.168.252.0
```

These let me inspect authorization, scope state, leases, reservations, and options. I also used DHCP Manager and compared server-side information with the client's `ipconfig /all` output.

### 8. Can a client have a valid DHCP lease and still have an Active Directory DNS problem?

Yes. In **SIM-DHCP-001**, I temporarily changed Option 006 to `1.1.1.1`.

The workstation retained its `.120` reservation and could reach the DC by IP. External DNS worked, but internal Northstar names failed. A direct query to `192.168.252.10` succeeded, isolating the issue to the DNS server supplied to the client.

I restored Option 006 to `.10`, renewed the client configuration, cleared its DNS cache, and verified internal resolution, DC discovery, and the secure channel again. This was an intentionally created support scenario.

### 9. Did every attempted fault produce the result you expected?

No. My first DNS-option test used VMware's `.2` resolver, and internal names and LDAP SRV queries still resolved according to my notes.

I recorded that result and used `1.1.1.1` to reproduce the intended failure. I did not invent a caching or forwarding explanation for the earlier successful queries.

That also remained separate from the Phase 5 pre-join observation, when the VMware resolver had failed to resolve the internal names.

### 10. What did you learn about interpreting DNS test output?

I need to inspect the returned record type rather than assume that any output proves the intended lookup succeeded.

In my recovery screenshot, the LDAP service-name query omitted `-Type SRV` and returned an SOA record in the Authority section. I do not present that as an SRV answer. My notes separately record explicit SRV validation using:

```powershell
Resolve-DnsName _ldap._tcp.dc._msdcs.ad.northstarsolutions.com -Type SRV
```

---

## 10. Time and Secure-Channel Troubleshooting

### 1. What unexpected issue did you encounter during Phase 7?

`Test-ComputerSecureChannel -Verbose` returned `False` during validation, even though DNS and DC discovery were working.

The workstation still reported domain membership, while `nltest /sc_verify` and `/sc_query` returned `NERR_Success`. I preserved those conflicting observations and investigated further instead of immediately treating the machine account as lost.

My investigation identified incorrect workstation time as a contributing condition. This was an unexpected lab incident, separate from the simulated DHCP DNS fault.

### 2. How did you inspect and validate time synchronization?

On the workstation, I used:

```powershell
Get-Date
Get-TimeZone
w32tm /query /source
w32tm /query /status
w32tm /stripchart /computer:ns-dc01.ad.northstarsolutions.com /samples:5 /dataonly
```

After correcting the clock, I used `w32tm /resync` and repeated the secure-channel test. The recovery output identified `NS-DC01` as the time source, showed closely synchronized offset samples, and returned `True` for the secure-channel check.

The observed recovery followed time correction. I would not claim that DHCP caused the issue or that the evidence proved time was its sole cause.

### 3. Would you immediately rejoin a computer to the domain after one failed secure-channel test?

I would first investigate the exact result, execution context, network configuration, DNS, DC discovery, and time state.

In this lab, the different tools did not all report failure. Recording those results and checking time helped me validate recovery without treating the first output as a complete diagnosis.

A trust repair or domain rejoin would need evidence supporting that action rather than being my automatic first step.

### 4. What did you do when Get-ADComputer was unavailable on the workstation?

The Active Directory PowerShell tools were not installed there, so the command was not recognized. I ran the directory query on `NS-DC01`, where the tools were available, and verified the enabled workstation object and Finance OU placement.

I treated the missing command as a local tooling limitation rather than evidence that Active Directory was unavailable.

---

## 11. Homelab Design and Production Considerations

### 1. How did you configure your Windows Server homelab?

I deployed Windows Server 2025 in VMware Workstation and standardized the server as `NS-DC01`.

Before deploying Active Directory, I inspected the VMware network, identified the subnet, gateway, and DHCP allocation range, and configured the server with the static address `192.168.252.10`.

I verified local gateway and external IP connectivity and established a pre-DNS baseline.

I then installed Active Directory Domain Services and DNS Server and promoted `NS-DC01` as the first Domain Controller in a new forest using `ad.northstarsolutions.com`.

After the restart, I validated the domain, forest, Global Catalog, AD DS and DNS services, and internal and external DNS resolution using Windows Server tools and PowerShell.

I then administered DNS records and reverse resolution, created the OU structure and employee accounts, configured departmental Global Security groups, and practiced a fictional employee identity lifecycle.

I configured the Finance workstation to use internal DNS, joined it to the domain, validated discovery and the secure channel, placed its computer object in the Finance Workstations OU, and checked a standard domain-user session.

I then validated computer and Finance user Group Policy, completed a simulated missing-link investigation, and migrated client addressing from VMware DHCP to Windows DHCP. I created a workstation reservation, investigated an unexpected time issue, and completed a simulated incorrect DNS-option scenario.

Phase 8 — File Services and Permissions is next. Resource-specific Domain Local groups and permissions remain future work.

### 2. Would you deploy only one Domain Controller in production?

Not necessarily.

My homelab currently uses one Domain Controller because I am working with limited compute resources and the purpose is hands-on learning.

A production environment would evaluate redundancy and availability requirements and would commonly use multiple Domain Controllers and DNS servers where appropriate.

I would not describe my single-DC homelab design as production high availability.

AD DS, DNS, and DHCP currently share `NS-DC01`, and I have not deployed a DHCP failover partner.

### 3. What production practices are you trying to apply in your homelab?

I am applying practices such as:

- Standardized naming
- Predictable infrastructure addressing
- Configuration baselines
- Pre-change and post-change validation
- Internal Active Directory DNS
- Least privilege
- Evidence-based troubleshooting
- Documentation
- Production-versus-homelab design considerations

I also document limitations rather than presenting a resource-constrained VMware environment as identical to a production enterprise deployment.

### 4. Which troubleshooting activities have you actually completed in these phases?

During Phases 4–5, I corrected an initial departmental membership mistake and observed the client's DNS prerequisite gap before joining the domain. I also completed the controlled Noah Wilson identity-lifecycle exercise.

In Phase 6, I completed `SIM-GPO-001`, a controlled missing Finance user OU link scenario. In Phase 7, I completed `SIM-DHCP-001`, a controlled incorrect DNS-option scenario, and investigated an unexpected workstation time and secure-channel issue.

I label those activities separately: validation corrections, administration exercises, simulated support incidents, and unexpected project incidents. I do not present the controlled faults as production outages or claim that future file-access scenarios are already complete.
