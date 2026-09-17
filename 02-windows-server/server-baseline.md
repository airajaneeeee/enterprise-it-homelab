# System Baseline — NS-DC01

## Purpose

This document records the technical baseline and infrastructure configuration of `NS-DC01`, the Windows Server 2025 system used in the Northstar Solutions enterprise IT homelab.

The baseline documents the server's identity, virtual hardware, network configuration, pre-deployment state, Active Directory Domain Services configuration, DNS configuration, directory administration, domain-client integration, centralized Group Policy, and Windows DHCP.

The current administration checkpoint reflects completed Phases 2–7 in the supplied lab notes and retained evidence. Earlier deployment observations remain historical records. Actions reported in the notes are distinguished from settings directly visible in the selected screenshots.

## System Identity

| Item | Value |
|---|---|
| VMware VM Name | `Northstar-DC01` |
| Windows Hostname | `NS-DC01` |
| FQDN | `NS-DC01.ad.northstarsolutions.com` |
| Operating System | Windows Server 2025 Standard Evaluation |
| Windows Version | 2009 |
| OS Build | 26100 |
| Platform | VMware Workstation |
| Current Role | Domain Controller / DNS Server / DHCP Server |
| Active Directory Domain | `ad.northstarsolutions.com` |
| NetBIOS Domain | `NORTHSTAR` |

## Virtual Hardware

| Component | Configuration |
|---|---|
| Processor | 2 vCPU |
| Memory | 4 GB |
| Virtual Disk | ~60 GB |
| Network Mode | VMware NAT / VMnet8 |

The virtual machine is intentionally resource-constrained for homelab use.

Production Domain Controllers would be sized according to workload, directory size, authentication demand, monitoring requirements, redundancy design, and organizational standards.

## VMware Network

| Setting | Value |
|---|---|
| Virtual Network | VMnet8 |
| Network Address | `192.168.252.0/24` |
| Subnet Mask | `255.255.255.0` |
| NAT / Default Gateway | `192.168.252.2` |
| VMware Host Adapter | `192.168.252.1` |
| VMware DHCP | Disabled during Phase 7 cutover |
| Current DHCP Server | `NS-DC01` — `192.168.252.10` |

The original VMware DHCP service used `192.168.252.254` with the range `192.168.252.128-192.168.252.254`. That configuration supported the earlier build stages documented below. Windows DHCP now supplies client leases; VMware NAT remains enabled.

## Initial Network Configuration

Before infrastructure services were deployed, `NS-DC01` received its network configuration through VMware DHCP.

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.129` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.254` |
| DNS Server | `192.168.252.2` |
| DNS Suffix | `localdomain` |
| DHCP Enabled | Yes |

The initial configuration was recorded before assigning the server a predictable infrastructure address.

## Static Network Configuration

`NS-DC01` was reconfigured with static IPv4 addressing before Active Directory and DNS deployment.

The table below records the later DNS client setting after Domain Controller promotion. During the original static-address configuration, the preferred DNS server was `192.168.252.10`; that earlier value remains visible in the historical screenshot and pre-deployment test output.

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.10` |
| Subnet Mask | `255.255.255.0` |
| Prefix Length | `/24` |
| Default Gateway | `192.168.252.2` |
| Preferred DNS | `127.0.0.1` |
| Alternate DNS | None |
| DHCP Enabled | No |

`NS-DC01` uses its own local DNS service because it hosts DNS for the Northstar Solutions Active Directory domain.

The current IPv4 DNS client setting is `127.0.0.1`, which is the IPv4 loopback address and points back to the DNS service running on the same server.

The server itself remains reachable on the network at:

```text
192.168.252.10
```

## Pre-Deployment Validation

Network connectivity and DNS behavior were tested before Active Directory Domain Services and DNS Server were deployed.

### Hostname

```cmd
hostname
```

Verified result:

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

At this stage, `NS-DC01` was configured to query its own address for DNS, but the DNS Server role had not yet been installed.

This established a pre-deployment DNS baseline.

## Active Directory Domain Services Configuration

The following Windows Server roles were installed:

- Active Directory Domain Services
- DNS Server

`NS-DC01` was then promoted as the first Domain Controller in a new Active Directory forest.

### Active Directory Configuration

| Item | Value |
|---|---|
| Forest Root Domain | `ad.northstarsolutions.com` |
| Active Directory Domain | `ad.northstarsolutions.com` |
| Distinguished Name | `DC=ad,DC=northstarsolutions,DC=com` |
| NetBIOS Domain | `NORTHSTAR` |
| Forest Functional Level | Windows Server 2025 |
| Domain Functional Level | Windows Server 2025 |
| Domain Controller | `NS-DC01.ad.northstarsolutions.com` |
| DNS Server | Enabled |
| Global Catalog | Enabled |
| Read-Only Domain Controller | No |

The default Active Directory database, log, and SYSVOL paths were used for this homelab deployment.

A Directory Services Restore Mode (DSRM) password was configured during promotion but is intentionally not recorded in project documentation.

## Domain Controller Validation

Post-deployment validation was performed after Domain Controller promotion and restart.

### Server Identity

```powershell
hostname
whoami
```

Verified results included:

```text
NS-DC01
NORTHSTAR\Administrator
```

This confirmed the server hostname and administrative domain context after promotion.

### Active Directory Domain

The domain configuration was inspected using:

```powershell
Get-ADDomain
```

Verified configuration included:

```text
DNSRoot               : ad.northstarsolutions.com
DistinguishedName     : DC=ad,DC=northstarsolutions,DC=com
DomainMode            : Windows2025Domain
Forest                : ad.northstarsolutions.com
Name                  : ad
NetBIOSName           : NORTHSTAR
PDCEmulator           : NS-DC01.ad.northstarsolutions.com
InfrastructureMaster  : NS-DC01.ad.northstarsolutions.com
RIDMaster             : NS-DC01.ad.northstarsolutions.com
```

### Active Directory Forest

The forest configuration was inspected using:

```powershell
Get-ADForest
```

Verified configuration included:

```text
Domains              : {ad.northstarsolutions.com}
ForestMode           : Windows2025Forest
GlobalCatalogs       : {NS-DC01.ad.northstarsolutions.com}
Name                 : ad.northstarsolutions.com
RootDomain           : ad.northstarsolutions.com
DomainNamingMaster   : NS-DC01.ad.northstarsolutions.com
SchemaMaster         : NS-DC01.ad.northstarsolutions.com
```

The server is currently located in the default Active Directory site:

```text
Default-First-Site-Name
```

## Active Directory and DNS Services

Service status was verified using:

```powershell
Get-Service NTDS,DNS
```

Verified state:

| Service | Function | Status |
|---|---|---|
| `NTDS` | Active Directory Domain Services | Running |
| `DNS` | DNS Server | Running |

This confirmed that both core services were operational after Domain Controller promotion.

## DNS Validation

DNS resolution was retested after Active Directory and DNS deployment.

### Internal DNS Resolution

```powershell
Resolve-DnsName NS-DC01.ad.northstarsolutions.com
```

The internal Domain Controller name successfully resolved to:

```text
192.168.252.10
```

This confirmed internal DNS resolution for the Northstar Solutions Active Directory namespace.

### External DNS Resolution

```powershell
Resolve-DnsName microsoft.com
```

The query successfully returned public IPv4 addresses for `microsoft.com`.

This confirmed that external DNS resolution was functioning through the DNS configuration on `NS-DC01`.

### DNS Forwarders

The DNS Server properties were reviewed.

No explicit DNS forwarders are currently configured.

External DNS resolution still succeeds because Windows DNS can use root hints when no forwarder is configured.

### IPv6 and Local DNS

During `nslookup` testing, the DNS server was displayed as:

```text
::1
```

`::1` is the IPv6 loopback address and refers to the local computer.

IPv6 was intentionally left enabled.

The presence of `::1` alone was not treated as a reason to disable IPv6.

### Before and After Comparison

| Test | Before DNS Deployment | After DNS Deployment |
|---|---|---|
| Gateway connectivity | Successful | Available |
| External IP connectivity | Successful | Available |
| Query DNS server at `192.168.252.10` | No response | Operational |
| Internal AD DNS resolution | Not available | Successful |
| External DNS resolution | Failed through `192.168.252.10` | Successful |

The comparison demonstrates the change from a server configured to query a DNS service that had not yet been deployed to an operational Active Directory DNS server capable of resolving both the internal domain namespace and external DNS names.

## DNS Record Administration State

The existing Active Directory DNS environment was inspected before additional records were created during Phase 4.

### Forward DNS and Service Discovery

The primary Active Directory namespace is:

```text
ad.northstarsolutions.com
```

The lab notes record inspection of the Domain Controller A record and AD-created DNS structures including `_msdcs`, `_sites`, `_tcp`, `_udp`, `DomainDnsZones`, and `ForestDnsZones`. SRV records for LDAP, Kerberos, Kerberos password operations, and the Global Catalog were reviewed.

### Reverse Lookup Zone

| Item | Recorded State |
|---|---|
| Network | `192.168.252.0/24` |
| Reverse Zone | `252.168.192.in-addr.arpa` |
| Zone Type | Primary |
| Active Directory-integrated | Yes |
| Dynamic Updates | Secure only |
| PTR Owner Name | `10.252.168.192.in-addr.arpa` |
| PTR Target | `ns-dc01.ad.northstarsolutions.com` |

The zone configuration is recorded in the lab notes. The retained DNS screenshot shows the A and PTR query results; it does not display the zone's integration or update-policy properties.

### Forward and Reverse Validation

Queries were run on `NS-DC01`, explicitly targeting its local DNS service:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Type A -Server 127.0.0.1 -DnsOnly
Resolve-DnsName 192.168.252.10 -Type PTR -Server 127.0.0.1 -DnsOnly
```

| Query | Returned Value | Displayed Section |
|---|---|---|
| A — `ns-dc01.ad.northstarsolutions.com` | `192.168.252.10` | Answer |
| PTR — `10.252.168.192.in-addr.arpa` | `ns-dc01.ad.northstarsolutions.com` | Answer |

The same screenshot also shows ordinary queries displaying `Section = Question`. This difference was retained as a validation observation. The Answer section alone is not treated as proof of the DNS authoritative-answer flag or as an explanation of how the ordinary queries obtained their results.

See [forward and reverse DNS evidence](screenshots/07-dns-forward-reverse-resolution-verification.png).

### Temporary Record Lifecycle

The notes record creation and validation of a temporary A record, `dns-lab.ad.northstarsolutions.com`, pointing to `192.168.252.10`, and a CNAME, `dns-alias.ad.northstarsolutions.com`, pointing to that temporary hostname.

Both training records were removed after validation. They are not part of the permanent DNS baseline. Creation and cleanup were documented without separate screenshots.

## Post-Deployment Administration Baseline

The following baseline checks were completed after AD DS and DNS deployment and before beginning Active Directory object administration.

| Item | Verified State |
|---|---|
| Network Profile | `DomainAuthenticated` |
| IPv4 Connectivity | Internet |
| IPv6 | Enabled |
| IPv6 Connectivity | `NoTraffic` observed during baseline check |
| Remote Desktop | Enabled |
| Network Level Authentication | Enabled |
| Remote Management | Enabled |
| Windows Defender Firewall | Enabled |
| ICMPv4 Echo Request | Inbound troubleshooting rule enabled |
| DNS Forwarders | None configured |
| External DNS Resolution | Successful |
| External Resolution Method | Root hints available |
| VMware Tools | Installed / operational |
| Time Zone | Verified |
| Windows Update | Verified |

The original notes record completion of the time-zone and Windows Update checks but do not retain the exact time-zone identifier or update result in this baseline.

The Windows Firewall remains enabled.

Rather than disabling the firewall, an ICMPv4 Echo Request inbound rule was enabled to support controlled connectivity testing and troubleshooting within the lab.

IPv6 was also kept enabled rather than disabled simply because the current Northstar environment primarily uses IPv4.

## Current Active Directory Administrative Structure

### Organizational Units

Northstar-managed directory objects are organized under a custom top-level OU:

```text
Northstar
├── Users
│   ├── Finance
│   ├── IT
│   └── Operations
├── Workstations
│   ├── Finance
│   ├── IT
│   └── Operations
├── Groups
├── Service Accounts
└── Disabled Objects
```

`NS-DC01` remains in the default `Domain Controllers` OU. The notes record accidental-deletion protection on the custom OUs; the [OU screenshot](screenshots/08-active-directory-ou-structure.png) demonstrates the hierarchy and DC placement, not that protection setting.

### Departmental Users

The six active fictional employee identities are:

| Name | Account | Department / Title | Users OU | Manager |
|---|---|---|---|---|
| Ava Chen | `ava.chen` | Finance / Financial Analyst | Finance | Daniel Kim |
| Daniel Kim | `daniel.kim` | Finance / Finance Manager | Finance | Not recorded |
| Maya Patel | `maya.patel` | IT / IT Support Technician | IT | Ethan Brooks |
| Ethan Brooks | `ethan.brooks` | IT / Systems Administrator | IT | Not recorded |
| Sofia Martinez | `sofia.martinez` | Operations / Operations Coordinator | Operations | Lucas Nguyen |
| Lucas Nguyen | `lucas.nguyen` | Operations / Operations Manager | Operations | Not recorded |

All departmental Users OUs in this table are beneath `Northstar > Users`. PowerShell output in the [user-verification screenshot](screenshots/09-active-directory-domain-users-verification.png) shows enabled accounts, account names, departments, titles, and Distinguished Names. Manager relationships are recorded in the lab notes but are not shown in that output. “Not recorded” does not assert that the directory attribute is empty.

The notes also record initial password-change-at-first-logon configuration. Temporary passwords are excluded from documentation, and that initial setting is not asserted as the current state after users have signed in.

### Departmental Security Groups

| Group | Recorded Scope / Category | Members |
|---|---|---|
| `GG-Finance-Users` | Global / Security | Ava Chen, Daniel Kim |
| `GG-IT-Users` | Global / Security | Maya Patel, Ethan Brooks |
| `GG-Operations-Users` | Global / Security | Sofia Martinez, Lucas Nguyen |

The groups implement Accounts → Global Groups from AGDLP. Resource-specific Domain Local groups and permissions have not yet been implemented.

Initial membership assignments were corrected during validation and then re-verified. The [group screenshot](screenshots/10-active-directory-security-groups-verification.png) clearly displays the departmental memberships; a desktop overlay partially obscures the scope/category output. Global/Security configuration is recorded in the notes.

The notes state that the ordinary IT employee accounts were not added to Domain Admins merely because of their job titles.

### Identity Lifecycle and Disabled Objects

The fictional account `noah.wilson` was used for an administration exercise:

| Stage | Title / Department | Manager | Location | Departmental Membership |
|---|---|---|---|---|
| Onboarding | Finance Assistant / Finance | Daniel Kim | `Northstar > Users > Finance` | `GG-Finance-Users` |
| Transfer | Operations Analyst / Operations | Lucas Nguyen | `Northstar > Users > Operations` | `GG-Operations-Users` |
| Offboarding | Operations attributes retained | Lucas Nguyen | `Northstar > Disabled Objects` | None |

Final recorded state:

```text
Name: Noah Wilson
SamAccountName: noah.wilson
Enabled: False
DistinguishedName: CN=Noah Wilson,OU=Disabled Objects,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
Displayed group membership: Domain Users
```

The [onboarding and transfer evidence](screenshots/11-active-directory-identity-lifecycle-offboarding-1.png) shows changes to title, department, OU, and group membership. The [offboarding evidence](screenshots/11-active-directory-identity-lifecycle-offboarding-2.png) shows the retained disabled object and removal of departmental memberships. Manager changes are recorded in the notes rather than the selected output.

This is a controlled identity-lifecycle exercise. The membership correction is a configuration-validation finding. No intentionally induced Phase 4 fault scenario is reported complete.

## Infrastructure Role

`NS-DC01` currently provides:

- Active Directory Domain Services
- Domain authentication infrastructure
- Internal DNS and Active Directory service discovery
- Global Catalog services
- Central directory services for users, groups, and computers
- Centralized computer and user Group Policy
- DHCP address allocation, scope options, and workstation reservation

Current architecture:

```text
Northstar Solutions
└── ad.northstarsolutions.com
    ├── NS-DC01
    │   ├── Windows Server 2025
    │   ├── Domain Controller / AD DS
    │   ├── DNS Server / Global Catalog
    │   ├── DHCP Server
    │   └── 192.168.252.10
    │
    └── NS-W11-01
        ├── Windows 11 Pro Finance workstation
        ├── Domain joined; Windows DHCP reservation → 192.168.252.120
        ├── Internal DNS via DHCP → 192.168.252.10
        └── Computer and Finance user Group Policy validated
```

## Domain Client Integration State

### Network Configuration Evolution

Before the domain join, the workstation was recorded as `WORKGROUP` with `CsPartOfDomain = False`.

| Setting | Pre-Join Observation | Phase 5 Domain Integration | Phase 7 Final State |
|---|---|---|---|
| Computer | `NS-W11-01` | `NS-W11-01` | `NS-W11-01` |
| Address Assignment | VMware DHCP | VMware DHCP retained | Windows DHCP reservation |
| Observed IPv4 Address | `192.168.252.128` | DHCP retained; `.128` also recorded before migration | `192.168.252.120` |
| Subnet Mask | `255.255.255.0` | No change reported | `255.255.255.0` |
| Default Gateway | `192.168.252.2` | No change reported | `192.168.252.2` |
| DHCP Server | `192.168.252.254` | VMware DHCP retained | `192.168.252.10` |
| IPv4 DNS Server | `192.168.252.2` | `192.168.252.10`, manually configured | `192.168.252.10`, supplied by Option 006 |

The pre-join address is a recorded DHCP observation, not a permanent address assignment. The workstation queries the DC's network address for DNS; the DC's loopback setting `127.0.0.1` is not used as the workstation's DNS server.

The notes record successful IP connectivity to the DC but failed Northstar domain and LDAP SRV resolution through VMware DNS. After configuring internal DNS, the workstation resolved the DC FQDN, domain, LDAP SRV records, and external names. This was a prerequisite validation observation, not a deliberately induced domain-join failure.

### Domain Membership and Trust

Validation on `NS-W11-01` used:

```powershell
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
nltest /dsgetdc:ad.northstarsolutions.com
Test-ComputerSecureChannel -Verbose
```

| Check | Observed Result |
|---|---|
| Computer | `NS-W11-01` |
| Domain | `ad.northstarsolutions.com` |
| Part of Domain | `True` |
| Discovered DC | `NS-DC01.ad.northstarsolutions.com` |
| DC Address | `192.168.252.10` |
| Secure Channel | `True`; reported in good condition |

These results are visible in the [administrative domain-membership validation](screenshots/12-ns-w11-01-domain-membership-verification.png).

### Active Directory Computer Object

The enabled workstation computer object was placed at:

```text
CN=NS-W11-01,OU=Finance,OU=Workstations,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
```

Server-side PowerShell validation used:

```powershell
Get-ADComputer NS-W11-01 -Properties DistinguishedName,Enabled | Select-Object Name,Enabled,DistinguishedName
```

The [computer-object evidence](screenshots/13-ns-w11-01-ad-computer-object-verification.png) shows `Enabled = True` and the Finance Workstations OU placement at Phase 5. The subsequent Phase 6 policy validation is recorded below; OU placement alone does not demonstrate policy application.

### Standard Domain-User Session

The [Finance session evidence](screenshots/14-domain-user-login-and-group-membership.png) shows:

- Authenticated identity `NORTHSTAR\ava.chen`.
- `NORTHSTAR\GG-Finance-Users` in the Windows security token.
- `USERDOMAIN=NORTHSTAR` and `USERDNSDOMAIN=AD.NORTHSTARSOLUTIONS.COM`.
- Successful Domain Controller discovery.
- No individual Ava Chen entry in local Administrators and no `BUILTIN\Administrators` entry in the displayed user token.

The output supports the standard-user session and departmental group membership, not a complete audit of all resource access. The secure-channel command at the bottom of this screenshot has no visible result; the successful result is retained in screenshot 12 from the administrative session.

## Centralized Group Policy State

### Workstation Computer Policy

| Item | Recorded State |
|---|---|
| GPO | `Northstar - Workstation Baseline` |
| Validation Computer | `NS-W11-01` |
| Computer OU | `Northstar > Workstations > Finance` |
| Applied Policy | Present in computer-scope `gpresult` |
| Registry Path | `HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System` |
| Registry Value | `InactivityTimeoutSecs` |
| Verified Value | `900` seconds |

Computer-scope validation used an elevated session on `NS-W11-01`:

```powershell
gpresult /h C:\gpo-report.html /scope computer
gpresult /r /scope computer
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System' -Name InactivityTimeoutSecs
```

The [workstation policy evidence](screenshots/15-workstation-baseline-gpo-verification.png) shows the named GPO in the applied computer policies and the configured inactivity value. The exact OU where the workstation GPO was linked is not established by the retained output.

### Finance User Policy

| Item | Recorded State |
|---|---|
| GPO | `Northstar - Finance User Policy` |
| Validation Identity | `NORTHSTAR\ava.chen` |
| User OU | `Northstar > Users > Finance` |
| Applied Policy | Present in user-scope `gpresult` |

Validation was performed in the Finance user's session:

```cmd
whoami
gpresult /r /scope user
```

The [Finance policy evidence](screenshots/16-finance-user-gpo-verification.png) confirms the identity and applied policy. The exact restriction setting is not specified in the supplied notes or selected evidence, so it is not asserted in this baseline.

### SIM-GPO-001 — Missing Finance OU Link

This was a controlled simulated support incident. The Finance policy stopped applying after its Finance user OU link was removed. The user remained in the correct OU, the GPO still existed, and its configured setting remained enabled, but `gpresult` no longer listed the GPO as applied.

The missing link was identified and restored. After `gpupdate /force`, user-scope results again listed `Northstar - Finance User Policy`. The notes also record restoration of the functional restriction.

The [simulated GPO recovery evidence](screenshots/17-simulated-gpo-link-troubleshooting.png) shows the absent applied policy, successful refresh, and restored policy result. It does not show the link edit or the restriction's user interface. The final baseline includes the restored Finance user OU link.

## Windows DHCP Configuration

### Address Plan

| Address / Range | Purpose |
|---|---|
| `192.168.252.0` | Network address |
| `192.168.252.1` | VMware host adapter |
| `192.168.252.2` | VMware NAT gateway |
| `192.168.252.10` | Static `NS-DC01` address; AD DS, DNS, and DHCP |
| `192.168.252.20`–`192.168.252.49` | Planned infrastructure range, excluded from DHCP allocation |
| `192.168.252.50`–`192.168.252.199` | Client allocation range, including the `.120` reservation |
| `192.168.252.120` | DHCP reservation for `NS-W11-01` |
| `192.168.252.200`–`192.168.252.254` | Outside the Windows scope; set aside for future lab use |
| `192.168.252.255` | Broadcast address |

The infrastructure exclusion is distinct from a DHCP reservation. The `.120` reservation assigns one address within the client range to the workstation's recorded MAC address.

### Installation and Authorization

The DHCP Server role was installed on `NS-DC01`. The notes record completion of post-installation configuration, creation of the DHCP Administrators and DHCP Users groups, and Active Directory authorization using `NORTHSTAR\Administrator`.

```powershell
Get-DhcpServerInDC
```

The [server-side DHCP evidence](screenshots/18-windows-dhcp-client-lease-verification.png) shows the authorized server as `192.168.252.10`, with DNS name `ns-dc01.ad.northstarsolutions.com`.

### IPv4 Scope and Options

| Setting | Recorded State |
|---|---|
| Scope Name | `Northstar Client Network` |
| Scope ID | `192.168.252.0` |
| Subnet Mask | `255.255.255.0` |
| Start Address | `192.168.252.20` |
| End Address | `192.168.252.199` |
| Exclusion | `192.168.252.20`–`192.168.252.49` |
| Lease Duration | 8 days |
| Final Scope State | Active |
| Option 003 — Router | `192.168.252.2` |
| Option 006 — DNS Servers | `192.168.252.10` |
| Option 015 — DNS Domain Name | `ad.northstarsolutions.com` |

Scope and option administration used:

```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.252.0
```

Screenshot 18 shows the scope range, subnet mask, eight-day duration, and transition from inactive to active. The full scope name is visible in screenshot 22. The exclusion and configured option values are recorded in the notes; screenshot 19 shows the resulting client gateway, DNS server, and suffix.

### VMware-to-Windows DHCP Cutover

The recorded cutover sequence was:

1. Create and configure the Windows scope while leaving it inactive.
2. Disable VMnet8's VMware DHCP service on the host while retaining NAT.
3. Activate the Windows DHCP scope on `NS-DC01`.
4. Configure `NS-W11-01` to obtain DNS automatically, with IPv4 addressing still automatic.
5. Release the previous lease, renew, and inspect the client configuration.
6. Inspect the lease on the Windows DHCP server.

Client commands used during cutover:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

Server-side validation used:

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.252.0
```

The first Windows DHCP lease for `NS-W11-01` was `192.168.252.50`, associated with client ID `00-0c-29-5f-31-87`. Screenshot 18 shows that lease as active. The [client configuration evidence](screenshots/19-windows-dhcp-client-configuration-verification.png) shows `.50`, DHCP enabled, DHCP and DNS server `.10`, gateway `.2`, and the Northstar connection-specific DNS suffix.

The workstation's earlier manually configured DNS setting was changed to automatic so this validation tested the DHCP-delivered DNS configuration.

### Finance Workstation Reservation

| Setting | Recorded State |
|---|---|
| Reservation Name | `NS-W11-01` |
| Reserved IPv4 Address | `192.168.252.120` |
| Client MAC Address | `00-0C-29-5F-31-87` |
| Description | Finance domain workstation |
| Supported Type | DHCP only |
| Client Addressing | Automatic; DHCP remains enabled |

The reservation was inspected using:

```powershell
Get-DhcpServerv4Reservation -ScopeId 192.168.252.0
```

The [reservation evidence](screenshots/22-dhcp-reservation-verification.png) shows `NS-W11-01` at `192.168.252.120` in DHCP Manager. The MAC address, description, and DHCP-only selection are recorded in the notes rather than displayed in that console view.

After lease release and renewal, the client received `.120`. The final client configuration is visible in screenshot 24, including the matching MAC address and `DHCP Enabled = Yes`. The server remains statically addressed at `.10`; the workstation uses a DHCP reservation.

## Phase 7 Validation and Troubleshooting

### DNS and Domain Validation

Checks on `NS-W11-01` included:

```powershell
Resolve-DnsName NS-DC01.ad.northstarsolutions.com
Resolve-DnsName _ldap._tcp.dc._msdcs.ad.northstarsolutions.com -Type SRV
Resolve-DnsName microsoft.com
nltest /dsgetdc:ad.northstarsolutions.com
Test-ComputerSecureChannel -Verbose
```

The notes record successful internal DNS, LDAP SRV discovery, Domain Controller discovery, and external DNS resolution after migration. The secure-channel discrepancy and recovery are retained below rather than presenting every intermediate check as successful.

### Unexpected Workstation Time Issue

The [initial investigation evidence](screenshots/20-secure-channel-time-skew-detection.png) shows:

| Check | Observed Result |
|---|---|
| `Test-ComputerSecureChannel -Verbose` | `False`; verbose output reported a broken secure channel |
| Domain Membership | `CsPartOfDomain = True` |
| `nltest /sc_verify:ad.northstarsolutions.com` | `NERR_Success` |
| `nltest /sc_query:ad.northstarsolutions.com` | `NERR_Success` |
| External DNS | Successful |
| Workstation Time Zone | `Pacific Standard Time` |

The notes additionally record working internal DNS, LDAP SRV resolution, and DC discovery. Investigation found incorrect workstation time as a contributing condition. The conflicting secure-channel results are preserved; they do not establish that the workstation had left the domain or that its computer account required replacement.

Time inspection and recovery included:

```powershell
Get-Date
Get-TimeZone
w32tm /query /status
w32tm /query /source
w32tm /resync
Test-ComputerSecureChannel -Verbose
```

After the clock was corrected and the client resynchronized, the [recovery evidence](screenshots/21-time-sync-secure-channel-recovery.png) shows successful resynchronization, `NS-DC01` as the time source, closely synchronized time-offset samples, and `Test-ComputerSecureChannel -Verbose` returning `True`.

This was a genuine unexpected lab incident discovered during DHCP validation, not an intentionally introduced fault. Recovery followed time correction; the evidence does not prove DHCP caused the issue or isolate time as its sole cause. The workstation time-zone output does not establish the server's exact time-zone setting.

The notes also record that `Get-ADComputer` was unavailable on the workstation because the Active Directory PowerShell tools were not installed there. The query was run on `NS-DC01`, where screenshot 18 shows the enabled computer object in its Finance Workstations OU. This was a tooling observation rather than an Active Directory service failure.

### SIM-DHCP-001 — Incorrect DNS Option

This was a controlled simulated support incident involving DHCP Option 006.

The first attempted fault used `192.168.252.2` as the DNS server. Internal names and LDAP SRV queries still resolved during that test, including direct DNS-only queries after cache clearing, according to the notes. That result did not reproduce the intended failure and is not assigned an unverified explanation. It is separate from the earlier Phase 5 pre-join DNS observation.

Option 006 was then changed to `1.1.1.1`, and the client renewed its lease and cleared its DNS cache.

| Check During Controlled Fault | Result |
|---|---|
| Client IPv4 Address | `192.168.252.120` |
| DHCP Server | `192.168.252.10` |
| Default Gateway | `192.168.252.2` |
| DHCP-delivered DNS Server | `1.1.1.1` |
| DC Reachability by IP | Successful, recorded in the notes |
| External DNS | Successful |
| Internal DC Name | Failed through the configured resolver |
| LDAP Service Discovery | Failed, recorded in the notes |
| Explicit DC Name Query to `192.168.252.10` | Successful |

The [controlled failure evidence](screenshots/23-simulated-dhcp-dns-option-failure.png) shows the public DNS client setting, successful external lookup, internal-name failures, and a successful direct query to the internal DNS server. The visible LDAP service-name query does not include `-Type SRV`; explicit SRV testing is reported in the notes.

The direct query used:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Server 192.168.252.10 -DnsOnly
```

Option 006 was restored to `192.168.252.10`. The client released and renewed its lease, then cleared its DNS cache:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /flushdns
ipconfig /all
```

The [recovery evidence](screenshots/24-simulated-dhcp-dns-option-recovery.png) shows the reserved address `.120`, DHCP and DNS server `.10`, gateway `.2`, successful DC name resolution, Domain Controller discovery, and a `True` secure-channel result.

The LDAP service-name query visible in screenshot 24 omits `-Type SRV` and returns an SOA record in the Authority section. It is not evidence of an SRV answer. Successful explicit SRV validation is recorded in the notes.

The final baseline has the internal DNS option restored. A valid lease and successful external resolution alone were insufficient to demonstrate correct Active Directory DNS configuration.

## Production Considerations

The current configuration is designed for a resource-constrained enterprise-style homelab.

A production Active Directory environment would typically evaluate requirements for:

- Multiple Domain Controllers
- DNS redundancy
- Server and service monitoring
- Active Directory backup and recovery
- Privileged administrative account separation
- Security baselines
- Centralized logging and auditing
- Network segmentation
- Formal IP address management
- Disaster recovery
- Change management

The single-Domain-Controller configuration used here does not provide infrastructure redundancy and should not be interpreted as a production high-availability design. AD DS, DNS, and DHCP share `NS-DC01` under the current host resource constraints. No DHCP failover partner is deployed.

## Baseline Status

**Windows Server Infrastructure Baseline — Complete through Phase 7: Windows DHCP**

The documented infrastructure baseline currently includes:

- Windows Server 2025 deployment
- Standardized server identity
- VMware network assessment
- Static IPv4 infrastructure addressing
- Pre-deployment connectivity and DNS assessment
- Active Directory Domain Services installation
- DNS Server installation
- New Active Directory forest and domain
- Domain Controller promotion
- Global Catalog configuration
- Active Directory domain and forest validation
- Active Directory and DNS service validation
- Internal DNS resolution
- External DNS resolution
- Domain-authenticated network profile validation
- Remote Desktop with Network Level Authentication
- Remote Management
- Windows Firewall retained with ICMPv4 Echo Request enabled for troubleshooting
- DNS forwarders reviewed; none configured
- Root-hints-based external DNS resolution confirmed
- IPv6 retained
- VMware Tools verification
- Time zone verification
- Windows Update baseline check
- Active Directory DNS record inspection
- Reverse lookup zone and PTR record administration
- DNS-only forward and reverse resolution validation
- Temporary A/CNAME administration and cleanup recorded in the lab notes
- Organizational Unit structure and Domain Controller placement
- Departmental user attributes, OU placement, and manager relationships
- Global Security groups and corrected departmental memberships
- Fictional employee onboarding, transfer, and offboarding
- Workstation internal DNS configuration with DHCP addressing retained
- Domain membership, DC discovery, and secure-channel validation
- Enabled computer object in the Finance Workstations OU
- Standard domain-user authentication and departmental group-token validation
- Computer-scoped workstation GPO and 900-second inactivity setting validation
- Finance user GPO application and simulated missing-link recovery
- DHCP address plan, exclusions, scope options, and eight-day leases
- DHCP Server installation and Active Directory authorization
- VMware DHCP retirement with VMware NAT retained
- Windows DHCP lease and automatic client DNS validation
- `NS-W11-01` reservation at `192.168.252.120`
- DHCP Manager and PowerShell administration
- Unexpected workstation time issue and secure-channel recovery validation
- Simulated incorrect DHCP DNS option diagnosis and recovery

Phases 2–7 are complete. Phase 8 — File Services and Permissions is next; resource-specific Domain Local groups and permissions remain future work for AGDLP. Later work includes PowerShell automation, server operations, security, monitoring, and additional integrated troubleshooting.

See the [server module overview](README.md), [screenshot guide](screenshots/README.md), and [workstation baseline](../01-windows-workstation/system-baseline.md) for related documentation. Supporting files are being brought to the same checkpoint separately.
