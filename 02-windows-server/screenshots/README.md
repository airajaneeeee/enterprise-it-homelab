# Windows Server Infrastructure — Screenshot Evidence

## Purpose

This directory contains selected visual evidence collected while deploying, configuring, and administering `NS-DC01` and integrating the Finance workstation `NS-W11-01` into the Northstar Solutions domain.

The evidence demonstrates server deployment, Active Directory and DNS validation, directory-object administration, a fictional identity lifecycle, workstation domain integration, centralized Group Policy, and Windows DHCP through Phase 7.

Screenshots are grouped by administrative activity rather than documented individually when several images support the same task.

---

## 1. Windows Server Identity and System Baseline

### Context

A Windows Server 2025 virtual machine was deployed to provide centralized infrastructure services for the Northstar Solutions environment.

Before Active Directory Domain Services and DNS were introduced, the server identity, operating system, virtual hardware, and initial network configuration were reviewed and documented.

### Evidence

![NS-DC01 server baseline](01-ns-dc01-server-baseline.png)

The server was standardized using the hostname:

`NS-DC01`

Naming convention:

- `NS` — Northstar Solutions
- `DC` — Domain Controller
- `01` — first server assigned to the role

The virtual machine was configured with:

- Windows Server 2025 Standard Evaluation
- 2 vCPU
- 4 GB RAM
- Approximately 60 GB virtual storage
- VMware Workstation
- VMware VMnet8 / NAT networking

### Outcome

The Windows Server system was deployed, standardized, and documented before infrastructure roles were introduced.

---

## 2. Static Infrastructure Addressing

### Context

`NS-DC01` initially received its network configuration dynamically through VMware DHCP.

Because the server was being prepared to provide core infrastructure services such as Active Directory and DNS, predictable addressing was configured before role deployment.

### Initial Network State

The server initially received:

- IPv4 address: `192.168.252.129`
- Subnet mask: `255.255.255.0`
- Default gateway: `192.168.252.2`
- DHCP server: `192.168.252.254`
- DNS server: `192.168.252.2`

The VMware network was identified as:

`192.168.252.0/24`

with the observed VMware DHCP allocation range:

`192.168.252.128-192.168.252.254`

### Configuration

![NS-DC01 static IP configuration](02-ns-dc01-static-ip-configuration.png)

The server was reconfigured with:

| Setting | Value |
|---|---|
| IPv4 Address | `192.168.252.10` |
| Subnet Mask | `255.255.255.0` |
| Prefix Length | `/24` |
| Default Gateway | `192.168.252.2` |
| Preferred DNS | `192.168.252.10` |
| Alternate DNS | None |
| DHCP | Disabled |

### Decision

A predictable IPv4 address was selected because systems providing core infrastructure services should be consistently reachable.

The address `192.168.252.10` is outside the VMware DHCP allocation range used by the lab.

`NS-DC01` was also configured to use its own address as its preferred DNS server in preparation for hosting DNS for the Active Directory environment.

After Domain Controller promotion, the server's current IPv4 DNS client configuration was later verified as `127.0.0.1`, which points to the DNS service running locally on `NS-DC01`. This later state is recorded in `server-baseline.md`; an additional screenshot was not retained because the existing evidence already demonstrates the server's DNS deployment and successful resolution.

### Validation

Connectivity was tested before Active Directory and DNS deployment using:

```cmd
ping 192.168.252.2
ping 8.8.8.8
```

The server successfully reached the VMware gateway and an external IP address.

A DNS query was also tested before DNS Server deployment:

```cmd
nslookup microsoft.com
```

At that stage, the query received no response because `NS-DC01` was already configured to query `192.168.252.10`, but no DNS Server role was running yet.

### Outcome

`NS-DC01` was configured with predictable infrastructure addressing and validated for IP connectivity before Active Directory deployment.

---

## 3. Active Directory Domain Services and DNS Installation

### Context

Northstar Solutions required centralized identity, authentication, DNS, and future policy management.

The Active Directory Domain Services and DNS Server roles were installed on `NS-DC01`.

### Evidence

![AD DS and DNS roles installed](03-ad-ds-dns-roles-installed.png)

Server Manager confirmed installation of:

- Active Directory Domain Services
- DNS Server
- Supporting Active Directory administration tools

After the roles were installed, the server still required promotion before it could operate as a Domain Controller.

### Domain Design

A new Active Directory forest was created using:

```text
ad.northstarsolutions.com
```

The NetBIOS domain name was configured as:

```text
NORTHSTAR
```

`NS-DC01` was promoted as the first Domain Controller in the new forest.

The promotion configuration included:

- Windows Server 2025 forest functional level
- Windows Server 2025 domain functional level
- DNS Server enabled
- Global Catalog enabled
- Read-Only Domain Controller disabled

A Directory Services Restore Mode password was configured during promotion but is intentionally not included in public project documentation.

### Outcome

The first Active Directory forest, domain, Domain Controller, and DNS Server for the Northstar Solutions environment were successfully deployed.

---

## 4. Active Directory Domain Validation

### Context

After Domain Controller promotion and restart, the Active Directory environment was validated rather than assuming that successful completion of the promotion wizard meant the infrastructure was fully operational.

### Evidence

![Active Directory domain verification](04-active-directory-domain-verification.png)

Validation commands included:

```powershell
hostname
whoami
Get-ADDomain
```

The results confirmed:

- Hostname: `NS-DC01`
- Administrative context: `NORTHSTAR\Administrator`
- DNS domain: `ad.northstarsolutions.com`
- Distinguished name: `DC=ad,DC=northstarsolutions,DC=com`
- NetBIOS domain: `NORTHSTAR`
- Domain functional level: Windows Server 2025
- Domain Controller: `NS-DC01.ad.northstarsolutions.com`

The domain output also identified `NS-DC01` as the current holder of domain-level FSMO roles returned by `Get-ADDomain`, including:

- PDC Emulator
- RID Master
- Infrastructure Master

### Outcome

`NS-DC01` was successfully operating as a Domain Controller in the Northstar Solutions Active Directory domain.

---

## 5. Active Directory Forest and Service Validation

### Context

Additional validation was performed to confirm the forest configuration, Global Catalog role, forest-level FSMO role holders, and status of the core Active Directory and DNS services.

### Evidence

![Active Directory forest and services verification](05-ad-forest-services-verification.png)

PowerShell validation included:

```powershell
Get-ADForest
Get-Service NTDS,DNS
```

The forest results confirmed:

- Forest root: `ad.northstarsolutions.com`
- Forest functional level: Windows Server 2025
- Root domain: `ad.northstarsolutions.com`
- Global Catalog: `NS-DC01.ad.northstarsolutions.com`
- Domain Naming Master: `NS-DC01.ad.northstarsolutions.com`
- Schema Master: `NS-DC01.ad.northstarsolutions.com`
- Active Directory site: `Default-First-Site-Name`

Service validation confirmed:

| Service | Function | Status |
|---|---|---|
| `NTDS` | Active Directory Domain Services | Running |
| `DNS` | DNS Server | Running |

### Interpretation

Because this is currently a single-Domain-Controller forest, `NS-DC01` holds the domain- and forest-level FSMO roles observed during validation.

The lab will examine FSMO administration in greater depth later in the Windows Server module.

### Outcome

The Active Directory forest, Global Catalog configuration, FSMO role placement, and core AD DS and DNS services were verified as operational.

---

## 6. DNS Resolution Validation

### Context

Before the DNS Server role was installed, `NS-DC01` had IP connectivity but could not obtain a DNS response from its own address because the DNS service did not yet exist.

After Active Directory and DNS deployment, name resolution was tested again.

### Evidence

![DNS resolution verification](06-dns-resolution-verification.png)

PowerShell testing included:

```powershell
Resolve-DnsName NS-DC01.ad.northstarsolutions.com
Resolve-DnsName microsoft.com
```

### Internal DNS Validation

The internal Domain Controller FQDN:

```text
NS-DC01.ad.northstarsolutions.com
```

successfully resolved to:

```text
192.168.252.10
```

This confirmed name resolution for the internal Active Directory namespace.

### External DNS Validation

The external name:

```text
microsoft.com
```

also resolved successfully and returned public IP addresses.

### Interpretation

The results demonstrated two separate DNS capabilities:

- Resolution of the internal Active Directory namespace
- Resolution of external DNS names

This provided a useful comparison with the pre-deployment state, where external IP connectivity was available but DNS queries to `192.168.252.10` received no response because the DNS Server role had not yet been installed.

### Outcome

Internal and external DNS resolution were successfully validated after DNS Server deployment.

---

## 7. Forward and Reverse DNS Administration

### Context

After the original DNS deployment, reverse DNS administration was added for `192.168.252.0/24`. The lab notes record an Active Directory-integrated reverse zone, `252.168.192.in-addr.arpa`, with secure dynamic updates and a PTR record for `NS-DC01`.

### Evidence

![NS-DC01 forward and reverse DNS query results](07-dns-forward-reverse-resolution-verification.png)

The screenshot shows queries executed on `NS-DC01` against its local DNS service:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Type A -Server 127.0.0.1 -DnsOnly
Resolve-DnsName 192.168.252.10 -Type PTR -Server 127.0.0.1 -DnsOnly
```

| Query | Returned Value | Displayed Section |
|---|---|---|
| A — `ns-dc01.ad.northstarsolutions.com` | `192.168.252.10` | Answer |
| PTR — `10.252.168.192.in-addr.arpa` | `ns-dc01.ad.northstarsolutions.com` | Answer |

An explicit `nslookup 192.168.252.10 127.0.0.1` query also returns the DC hostname.

### Interpretation

The ordinary queries shown below the explicit tests display `Section = Question`. This difference is retained as a validation observation; the screenshot alone does not establish how those ordinary queries obtained their results.

The explicit queries demonstrate the expected A and PTR results. The Answer section alone is not treated as proof of the DNS authoritative-answer flag. Zone integration, secure-update settings, and temporary A/CNAME cleanup are recorded in the lab notes rather than displayed by this image.

### Outcome

Forward and reverse resolution for `NS-DC01` were validated with DNS-only queries explicitly targeting the local DNS service.

---

## 8. Active Directory Organizational Unit Structure

### Context

A custom OU hierarchy was created to support user and workstation administration, future Group Policy targeting, and identity lifecycle management.

### Evidence

![Northstar OU hierarchy and Domain Controller placement](08-active-directory-ou-structure.png)

The visible hierarchy includes:

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

The default `Domain Controllers` OU is selected and contains `NS-DC01`. The notes record accidental-deletion protection for the custom OUs, but that setting is not visible in this hierarchy view.

### Outcome

The directory has an administrative structure for the later user, workstation, identity lifecycle, and policy tasks. The Domain Controller remains in its default OU.

---

## 9. Domain User Administration

### Context

Six fictional employees were created across Finance, IT, and Operations. Directory attributes and departmental OU placement were checked with PowerShell.

### Evidence

![Enabled Northstar domain users, attributes, and Distinguished Names](09-active-directory-domain-users-verification.png)

The displayed results show:

| Employee | Account | Department | Title |
|---|---|---|---|
| Ava Chen | `ava.chen` | Finance | Financial Analyst |
| Daniel Kim | `daniel.kim` | Finance | Finance Manager |
| Maya Patel | `maya.patel` | IT | IT Support Technician |
| Ethan Brooks | `ethan.brooks` | IT | Systems Administrator |
| Sofia Martinez | `sofia.martinez` | Operations | Operations Coordinator |
| Lucas Nguyen | `lucas.nguyen` | Operations | Operations Manager |

Each account is shown as enabled. A second query displays Distinguished Names identifying the departmental Users OUs beneath `Northstar`.

Manager relationships and initial password-change settings are recorded in the lab notes; they are not displayed in this screenshot. Temporary passwords are excluded from documentation.

### Outcome

The six enabled employee accounts, departmental attributes, and OU placement were verified for later authentication, access-control, and policy exercises.

---

## 10. Active Directory Security Groups

### Context

Departmental Global Security groups were created to organize account membership before resource-specific permissions are introduced.

### Evidence

![Northstar departmental security-group membership verification](10-active-directory-security-groups-verification.png)

The `Get-ADGroupMember` results clearly show:

| Group | Members |
|---|---|
| `GG-Finance-Users` | Ava Chen, Daniel Kim |
| `GG-IT-Users` | Maya Patel, Ethan Brooks |
| `GG-Operations-Users` | Sofia Martinez, Lucas Nguyen |

A desktop overlay partially obscures the group scope/category output at the bottom. The Global/Security configuration is recorded in the lab notes; the retained image primarily supports the membership verification.

### Validation Finding

The notes record incorrect initial group assignments discovered during verification. The assignments were corrected and re-verified. This image shows the corrected memberships, not the earlier incorrect state.

The finding is documented as a configuration-validation correction rather than an intentionally induced support incident.

### Outcome

Departmental memberships were verified. This establishes the Accounts → Global Groups portion of AGDLP; Domain Local groups and resource permissions remain future work.

---

## 11. Identity Lifecycle Administration

### Context

The fictional account `noah.wilson` was used to practice onboarding, an internal transfer, and offboarding. Both images retain their existing filenames and belong to this single activity.

### Onboarding and Transfer Evidence

![Noah Wilson onboarding and transfer attributes, OU placement, and group membership](11-active-directory-identity-lifecycle-offboarding-1.png)

The first image shows the account enabled in both stages:

| Stage | Title / Department | Users OU | Displayed Memberships |
|---|---|---|---|
| Onboarding | Finance Assistant / Finance | Finance | `Domain Users`, `GG-Finance-Users` |
| Transfer | Operations Analyst / Operations | Operations | `Domain Users`, `GG-Operations-Users` |

The results demonstrate separate updates to attributes, OU placement, and group membership. Manager changes are recorded in the notes but are not displayed in the selected output.

### Offboarding Evidence

![Noah Wilson disabled account, Disabled Objects OU, and remaining group membership](11-active-directory-identity-lifecycle-offboarding-2.png)

The final results show:

- `Enabled = False`.
- Operations title and department retained.
- Distinguished Name under `OU=Disabled Objects,OU=Northstar`.
- Only `Domain Users` in the displayed group-membership results.

### Outcome

The fictional account was onboarded, transferred, and retained as a disabled directory object after departmental memberships were removed. This is a controlled administration exercise, not a real employee offboarding or an unexpected outage.

---

## 12. Windows 11 Domain Membership and Secure Channel

### Context

The lab notes record configuration of `NS-W11-01` to use internal DNS at `192.168.252.10` while retaining VMware DHCP addressing. The workstation was then joined to the Northstar domain.

### Evidence

![NS-W11-01 domain membership, Domain Controller discovery, and secure-channel validation](12-ns-w11-01-domain-membership-verification.png)

The administrative validation session shows:

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

This image shows post-join results. The pre-join DNS observations and DNS-client setting change are recorded in the notes rather than shown here.

### Outcome

Domain membership, DC discovery, and the workstation-domain secure channel were verified beyond the domain-join wizard.

---

## 13. Active Directory Computer Object

### Context

After the domain join, the workstation computer account was placed in the Finance Workstations OU to support future policy targeting.

### Evidence

![Enabled NS-W11-01 computer object in the Finance Workstations OU](13-ns-w11-01-ad-computer-object-verification.png)

The server-side query shows the account enabled at:

```text
CN=NS-W11-01,OU=Finance,OU=Workstations,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
```

The Active Directory Users and Computers view also shows `NS-W11-01` under the selected Finance Workstations OU.

### Outcome

The enabled computer object is in the intended administrative location. This demonstrates placement, not deployment or application of a new Group Policy.

---

## 14. Standard Domain User Authentication

### Context

A Finance user session was checked on the domain-joined workstation to validate the authenticated identity, departmental membership, and standard-user context.

### Evidence

![Ava Chen domain identity, group token, local Administrators membership, and DC discovery](14-domain-user-login-and-group-membership.png)

The screenshot shows:

- `whoami` identifying `NORTHSTAR\ava.chen`.
- The user's SID from `whoami /user`.
- `NORTHSTAR\GG-Finance-Users` in `whoami /groups` output.
- `USERDOMAIN=NORTHSTAR` and `USERDNSDOMAIN=AD.NORTHSTARSOLUTIONS.COM`.
- Local Administrators membership with no individual Ava Chen entry.
- No `BUILTIN\Administrators` entry in the displayed user token.
- Successful Domain Controller discovery using `nltest`.

### Interpretation

The token and local group output support the documented standard-user session and departmental membership. They do not constitute a complete audit of resource permissions.

`Test-ComputerSecureChannel` appears at the bottom without a visible result. The successful secure-channel result is documented in section 12 from the separate administrative session; this image is not used to claim that the command succeeded in Ava's session.

### Outcome

The workstation authenticated the Finance domain identity and reflected the departmental group in the Windows session.

---

## 15. Workstation Group Policy Validation

### Context

Phase 6 introduced the computer-scoped `Northstar - Workstation Baseline` policy. Its application and workstation inactivity setting were checked on `NS-W11-01` in an elevated session.

### Evidence

![Workstation computer policy and 900-second inactivity value](15-workstation-baseline-gpo-verification.png)

The visible commands include:

```powershell
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System' -Name InactivityTimeoutSecs
gpresult /h C:\gpo-report.html /scope computer
gpresult /r /scope computer
```

The output shows:

- `InactivityTimeoutSecs` returning `900` seconds.
- The computer object in `Northstar > Workstations > Finance`.
- Policy applied from `NS-DC01.ad.northstarsolutions.com`.
- `Northstar - Workstation Baseline`, Default Domain Policy, and Local Group Policy in the applied computer-policy list.

### Interpretation

The screenshot verifies the applied computer policy and registry value. It does not display the GPO's exact link location or a timed lock-screen test. The HTML-report command is visible, but the report contents are not shown.

### Outcome

Computer-scope policy application and the 900-second inactivity value were verified on the Finance workstation.

---

## 16. Finance User Group Policy Validation

### Context

The Finance user policy was checked separately from computer policy, in Ava Chen's domain session on `NS-W11-01`.

### Evidence

![Finance user identity and applied user policy](16-finance-user-gpo-verification.png)

The screenshot shows:

- `gpresult /r /scope user` for `NORTHSTAR\ava.chen`.
- Ava's Distinguished Name under `Northstar > Users > Finance`.
- `Northstar - Finance User Policy` in the applied policy list.
- Local Group Policy filtered out as empty.
- `GG-Finance-Users` in the displayed group membership.
- `whoami` confirming `northstar\ava.chen`.

### Interpretation

The output demonstrates user-policy application in the intended Finance session. It does not identify the exact restriction setting configured in the GPO.

### Outcome

The Finance user policy was verified independently of the workstation's computer policy.

---

## 17. Simulated Missing GPO Link Troubleshooting

### Context

**SIM-GPO-001** was a controlled simulated support incident. The notes record a missing Finance user OU link while the user remained in the correct OU and the GPO and its enabled setting still existed.

### Evidence

![Missing applied Finance policy followed by refresh and restored application](17-simulated-gpo-link-troubleshooting.png)

The visible sequence includes:

1. An earlier applied-policy list displaying `N/A`.
2. `gpupdate /force` reporting successful computer and user policy updates.
3. `gpresult /r /scope user` again listing `Northstar - Finance User Policy` for Ava Chen.

### Interpretation

The image demonstrates the before-and-after policy results. Link diagnosis, link restoration, and the restored functional restriction are recorded in the notes; the screenshot does not show those actions directly. Policy refresh followed the link correction.

### Outcome

Finance user-policy application was restored after the OU link was repaired. This was a simulated support incident, not an unexpected project outage.

---

## 18. Windows DHCP Authorization, Scope, and Lease Validation

### Context

Phase 7 moved client address allocation from VMware DHCP to Windows DHCP on `NS-DC01`. The notes record disabling VMware DHCP before activating the Windows scope, while retaining VMware NAT.

### Evidence

![Authorized Windows DHCP server, scope activation, and client lease](18-windows-dhcp-client-lease-verification.png)

The server-side output shows:

| Check | Visible Result |
|---|---|
| `Get-DhcpServerInDC` | `192.168.252.10`, `ns-dc01.ad.northstarsolutions.com` |
| Scope ID / Mask | `192.168.252.0` / `255.255.255.0` |
| Scope Range | `192.168.252.20`–`192.168.252.199` |
| Lease Duration | `8.00:00:00` — eight days |
| Scope State | Inactive in the earlier query; Active in the later query |
| Workstation Lease | `192.168.252.50`, client ID `00-0c-29-5f-31-87`, Active |

The lease table also contains another active lease at `.51`; it is not identified as the Finance workstation. A subsequent `Get-ADComputer` query shows `NS-W11-01` enabled in the Finance Workstations OU.

### Interpretation

The screenshot supports server authorization, scope activation, and lease validation. The scope name is truncated here and is visible in full in section 22. Exclusions, scope options, and the host-side VMware DHCP change are recorded in the notes rather than displayed in this image.

### Outcome

The authorized Windows DHCP server had an active scope and an active lease for the Finance workstation.

---

## 19. Windows DHCP Client Configuration

### Context

The workstation retained automatic IPv4 addressing and was changed to obtain DNS automatically so validation would test the Windows DHCP scope options.

### Evidence

![Workstation lease renewal and Windows DHCP network configuration](19-windows-dhcp-client-configuration-verification.png)

The screenshot shows `ipconfig /release`, `ipconfig /renew`, and `ipconfig /all` with:

| Setting | Visible Value |
|---|---|
| Hostname | `NS-W11-01` |
| Physical Address | `00-0C-29-5F-31-87` |
| DHCP Enabled | Yes |
| IPv4 Address | `192.168.252.50` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.10` |
| DNS Server | `192.168.252.10` |
| Connection-specific DNS Suffix | `ad.northstarsolutions.com` |

### Interpretation

This is the initial Windows DHCP lease, before the later `.120` reservation. The `.50` address remains part of the build history. The client output shows the delivered values; the server-side option configuration is documented in the baseline and lab notes.

### Outcome

The workstation received its network configuration from Windows DHCP while continuing to use VMware's NAT gateway.

---

## 20. Unexpected Secure-Channel and Time Investigation

### Context

An unexpected secure-channel test failure was investigated during Phase 7 validation. This was separate from the controlled DHCP DNS-option simulation.

### Evidence

![Conflicting secure-channel results, domain membership, and workstation clock inspection](20-secure-channel-time-skew-detection.png)

The screenshot shows:

- `Test-ComputerSecureChannel -Verbose` returning `False` and reporting a broken secure channel.
- Successful external DNS resolution.
- `CsPartOfDomain = True` for `ad.northstarsolutions.com`.
- `nltest /sc_verify` and `/sc_query` reporting `NERR_Success`.
- `Get-Date` and `Get-TimeZone`, with the workstation time-zone ID `Pacific Standard Time`.

### Interpretation

The conflicting test results are preserved. This image does not show that the workstation left the domain or that DHCP caused the failure. The notes identify incorrect workstation time as a contributing condition; this screenshot alone does not measure the clock offset from the DC.

### Outcome

The discrepancy prompted further time-synchronization checks and recovery validation rather than treating a single failed test as a complete diagnosis.

---

## 21. Time Synchronization and Secure-Channel Recovery

### Context

The notes record correcting the workstation clock and resynchronizing with `NS-DC01` before repeating validation.

### Evidence

![Successful time synchronization and repeated secure-channel validation](21-time-sync-secure-channel-recovery.png)

The output shows:

- `w32tm /resync` completing successfully.
- `w32tm /query /source` identifying `NS-DC01.ad.northstarsolutions.com`.
- Time status identifying source IP `192.168.252.10`.
- Five strip-chart offset samples of approximately `+0.0017` to `+0.0019` seconds.
- Two `Test-ComputerSecureChannel -Verbose` results returning `True` and reporting good condition.
- The workstation time-zone ID `Pacific Standard Time`.

The visible offset check used:

```cmd
w32tm /stripchart /computer:ns-dc01.ad.northstarsolutions.com /samples:5 /dataonly
```

### Interpretation

The client was closely synchronized with the DC during these samples. Secure-channel validation succeeded after time correction and resynchronization. This supports the recovery sequence without establishing time as the sole cause of the earlier discrepancy.

### Outcome

Time synchronization and successful secure-channel validation were recorded after the unexpected lab issue.

---

## 22. Finance Workstation DHCP Reservation

### Context

A DHCP reservation was created to give `NS-W11-01` a predictable address while retaining automatic client configuration.

### Evidence

![NS-W11-01 reservation at 192.168.252.120 in DHCP Manager](22-dhcp-reservation-verification.png)

DHCP Manager shows:

- Server: `ns-dc01.ad.northstarsolutions.com`.
- IPv4 scope: `[192.168.252.0] Northstar Client Network`.
- Reservation: `[192.168.252.120] NS-W11-01`.

### Interpretation

The console view demonstrates the reservation name and address. Its MAC address, description, and DHCP-only selection are recorded in the notes rather than visible here. Client use of `.120` is shown in sections 23–24.

Server Manager event entries are visible behind DHCP Manager. Their details and resolution are not shown, so this image is not used to claim a clean event log or diagnose those entries.

### Outcome

The Finance workstation reservation was recorded in the Windows DHCP scope. The workstation continued to use DHCP rather than a manually assigned static address.

---

## 23. Simulated Incorrect DHCP DNS Option

### Context

**SIM-DHCP-001** was a controlled simulated support incident. DHCP Option 006 was temporarily changed to `1.1.1.1` to reproduce internal DNS failure on a domain client.

The notes record an earlier test using `192.168.252.2` that still resolved internal names and did not reproduce the intended failure. That test is not shown in this image.

### Evidence

![Public DNS client setting, internal lookup failures, and successful direct internal DNS query](23-simulated-dhcp-dns-option-failure.png)

The client configuration shows:

| Setting | Visible Value |
|---|---|
| IPv4 Address | `192.168.252.120` |
| DHCP Enabled | Yes |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.10` |
| DNS Server | `1.1.1.1` |

The adjacent PowerShell output shows successful external resolution, failed queries for the internal DC and LDAP service name, and a successful DC A-record query explicitly targeting `192.168.252.10` with `-DnsOnly`.

### Interpretation

The successful direct internal query supports isolation to the client's DNS configuration. The visible LDAP service-name query omits `-Type SRV`; explicit SRV testing and IP-connectivity checks are recorded in the notes.

### Outcome

The controlled fault demonstrated that a valid lease and working external DNS do not establish correct internal Active Directory DNS configuration.

---

## 24. Simulated DHCP DNS-Option Recovery

### Context

Option 006 was restored to `192.168.252.10`. The notes record lease release and renewal, DNS cache clearing, and renewed domain validation to close `SIM-DHCP-001`.

### Evidence

![Restored internal DNS configuration, DC discovery, and secure-channel validation](24-simulated-dhcp-dns-option-recovery.png)

The screenshot shows lease renewal, cache clearing, and client configuration with:

- Reserved IPv4 address `192.168.252.120` and MAC `00-0C-29-5F-31-87`.
- `DHCP Enabled = Yes`.
- DHCP and DNS server `192.168.252.10`.
- Gateway `192.168.252.2` and the Northstar connection-specific DNS suffix.
- Successful DC A-record resolution through both an explicit internal query and an ordinary query.
- Successful Domain Controller discovery using `nltest /dsgetdc`.
- `Test-ComputerSecureChannel -Verbose` returning `True`.

### Interpretation

The visible LDAP service-name query omits `-Type SRV` and returns an SOA record in the Authority section. It is not an SRV answer. Successful explicit SRV validation is recorded separately in the notes.

### Outcome

The internal DNS client configuration was restored, and DC name resolution, discovery, and secure-channel validation succeeded. The reservation remained active throughout the simulated DNS-option failure and recovery.

---

## Evidence Summary

The selected screenshots demonstrate server administration and workstation integration across the following areas:

- Windows Server deployment and identity standardization
- Windows Server virtual hardware baseline
- VMware network assessment
- Static IPv4 infrastructure addressing
- Pre-deployment network validation
- Active Directory Domain Services installation
- DNS Server installation
- New Active Directory forest creation
- Active Directory domain creation
- Domain Controller promotion
- Global Catalog configuration
- Active Directory domain validation
- Active Directory forest validation
- FSMO role observation
- Active Directory Domain Services verification
- DNS Server service verification
- Internal DNS resolution
- External DNS resolution
- DNS-only forward and reverse record validation
- Active Directory OU structure and Domain Controller placement
- Enabled employee identities, directory attributes, and OU placement
- Departmental group membership verification
- Fictional employee onboarding, transfer, and offboarding
- Workstation domain membership and Domain Controller discovery
- Workstation-domain secure-channel validation
- Enabled workstation computer object and Finance OU placement
- Domain-user identity, group-token membership, and standard-user session checks
- Computer-scoped Group Policy and the 900-second inactivity setting
- Finance user-policy application and simulated missing-link recovery
- DHCP authorization, scope activation, and client lease validation
- Windows DHCP client configuration and workstation reservation
- Unexpected secure-channel discrepancy, time synchronization, and recovery
- Simulated incorrect DHCP DNS-option failure isolation and recovery

The evidence is intentionally selective.

Not every wizard page, command, or administrative action requires its own screenshot.

Detailed technical configuration is maintained in `server-baseline.md`, reusable concepts and procedures are maintained in the Windows Server Knowledge Base, and troubleshooting evidence will be documented separately as genuine project incidents, simulated support incidents, or focused troubleshooting exercises.

See the [server baseline](../server-baseline.md), [server knowledge base](../../knowledge-base/windows-server-administration.md), and [module overview](../README.md) for supporting documentation. Written lab notes cover actions without dedicated images; each section above distinguishes those actions from visible evidence.

Additional screenshots should only be added when they demonstrate a new administrative capability, important configuration change, meaningful validation result, or genuine troubleshooting activity.

## Status

**Windows Server Infrastructure Screenshot Evidence — Complete through Phase 7**

Current evidence covers:

- Server deployment
- Static networking
- AD DS installation
- DNS installation
- Domain Controller promotion
- Active Directory validation
- DNS validation
- DNS record administration and reverse resolution
- Organizational Units, domain users, and departmental memberships
- Identity lifecycle administration
- Windows 11 domain integration and computer-object placement
- Domain-user authentication and workstation trust validation
- Centralized computer and user Group Policy
- Simulated missing Finance GPO link recovery
- Windows DHCP deployment, client configuration, and reservation
- Unexpected time issue investigation and secure-channel recovery
- Simulated DHCP DNS-option troubleshooting

The retained set contains 25 images across activities 01–24, including the two images for activity 11. Existing filenames and historical screenshots are preserved.

Phases 6–7 are complete. Phase 8 — File Services and Permissions is next. The next meaningful screenshot will use the `25-...` prefix.

Later evidence will be added selectively for file services, PowerShell automation, server operations, security, monitoring, and troubleshooting. The completed Phase 6–7 simulations are labeled `SIM-GPO-001` and `SIM-DHCP-001`; the unexpected time issue is documented separately from those controlled scenarios.
