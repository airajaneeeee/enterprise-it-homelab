# System Baseline — NS-W11-01

## Purpose

This document records the technical baseline configuration of `NS-W11-01`, the Windows 11 Pro Finance workstation used in the Northstar Solutions enterprise IT homelab.

The baseline preserves the workstation's original standalone identity, hardware, local accounts, networking, storage, policy, and security observations. Separate checkpoints record Phase 5 domain integration, Phase 6 centralized Group Policy, and Phase 7 Windows DHCP and troubleshooting validation.

Original configuration tables describe the standalone baseline unless identified as part of a later checkpoint. The later notes do not establish a fresh hardware, storage, update, or comprehensive endpoint-security assessment.

## System Identity

| Item | Value |
|---|---|
| VMware VM Name | `Northstar-W11-Client01` |
| Windows Hostname | `NS-W11-01` |
| Operating System | Windows 11 Pro |
| Windows Version | 25H2 |
| OS Build | 26200.9445 |
| Platform | VMware Workstation |
| Assigned Role | Finance Department Workstation |

## Virtual Hardware

| Component | Value |
|---|---|
| Processor / vCPU | 2 vCPUs — 12th Gen Intel(R) Core(TM) i7-12700H (2.69 GHz) |
| Installed RAM | 8.00 GB |
| System Type | 64-bit operating system, x64-based processor |

These values are retained from the original workstation assessment. Later planning notes refer to approximately 4 GB of workstation RAM, but a subsequent allocation change has not been confirmed. The historical 8 GB observation is therefore preserved; current allocation should be verified before making capacity-dependent changes.

## Workstation Identity Configuration

### Original State

The workstation initially used the Windows computer name created during the original setup.

### Configuration Change

The workstation was renamed to:

`NS-W11-01`

### Naming Convention

- `NS` = Northstar Solutions
- `W11` = Windows 11 workstation
- `01` = Workstation 01

### Verification

The workstation hostname was verified using:

```cmd
hostname
```

Verified result:

```text
NS-W11-01
```

The Windows **Settings > System > About** page was also reviewed to confirm the workstation hostname, operating system edition, Windows version, OS build, installed memory, processor information, and system architecture.

## Local User Configuration

Two dedicated local accounts were configured for the workstation.

| Account | Purpose | Access Level |
|---|---|---|
| `labadmin` | IT administration | Local Administrator |
| `finance.user` | Simulated Finance employee | Standard User |

### Administrative Account

`labadmin` was added to the local `Administrators` group and is used for privileged IT administration tasks.

Administrative context and local administrator membership were inspected using:

```cmd
whoami
net localgroup administrators
```

### Standard Employee Account

`finance.user` was configured as a standard user and was not added to the local `Administrators` group.

This configuration separates normal employee activity from privileged administrative operations and supports the principle of least privilege.

Administrative operations performed from the standard-user context require authorized elevation rather than permanent administrator membership.

## Network Baseline

At the standalone baseline, `NS-W11-01` was connected to the VMware virtual network through `Ethernet0` and received its IPv4 configuration dynamically through DHCP. The table preserves that earlier state, including VMware DNS.

| Setting | Value |
|---|---|
| Network Adapter | `Ethernet0` |
| Adapter Description | Intel(R) 82574L Gigabit Network Connection |
| IPv4 Address | `192.168.252.128` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.254` |
| DNS Server | `192.168.252.2` |
| DNS Suffix | `localdomain` |
| DHCP Enabled | Yes |

Detailed network configuration was inspected using:

```cmd
ipconfig /all
```

DNS and DHCP administration and troubleshooting commands practiced during the workstation module included:

```cmd
nslookup
ipconfig /displaydns
ipconfig /flushdns
ipconfig /release
ipconfig /renew
```

The workstation was DHCP-configured at this stage. The recorded IPv4 address is a lease observation, not a permanent address assignment.

During the later domain-integration checkpoint below, DHCP addressing was retained and the IPv4 DNS server was changed to `192.168.252.10`. The historical `192.168.252.2` DNS value and `localdomain` suffix above are not presented as the post-join configuration.

## Local Policy Configuration

Local Group Policy was reviewed and configured using:

```cmd
gpedit.msc
```

The following workstation policy was configured:

**Do not allow passwords to be saved**

for the Remote Desktop Connection client.

The policy change was applied using:

```cmd
gpupdate /force
```

Group Policy diagnostic tools reviewed during the workstation module included:

```cmd
gpresult /r
rsop.msc
```

This local policy exercise introduced standalone Windows policy administration before the centralized domain policies documented in the Phase 6 checkpoint below.

## Storage Configuration

| Volume | Purpose | File System | Capacity | Status |
|---|---|---|---|---|
| `C:` | Windows operating system | NTFS | 952.60 GB | Healthy |
| `Finance-Data (F:)` | Simulated Finance department data | NTFS | ~10 GB | Healthy |

A secondary virtual disk was provisioned and configured using Windows Disk Management.

The disk was initialized, formatted using NTFS, assigned drive letter `F:`, and labeled:

`Finance-Data`

Read/write access to the new volume was verified through Windows File Explorer.

## Endpoint Security Baseline

| Control | Assessment |
|---|---|
| Microsoft Defender Antivirus | Verified Active |
| Real-Time Protection | Verified Enabled |
| Windows Defender Firewall | Verified Enabled |
| User Account Control | Tested |
| Least-Privilege User Configuration | Implemented |
| BitLocker | Protection Off |

Microsoft Defender and Windows Firewall configuration were reviewed using Windows Security, Windows administrative tools, and PowerShell.

Relevant PowerShell commands included:

```powershell
Get-MpComputerStatus
Get-NetFirewallProfile
Get-NetConnectionProfile
```

### BitLocker Assessment

BitLocker status was assessed using:

```cmd
manage-bde -status
```

The operating-system volume reported:

| Item | Result |
|---|---|
| Conversion Status | Fully Decrypted |
| Percentage Encrypted | 0.0% |
| Protection Status | Protection Off |
| Lock Status | Unlocked |
| Key Protectors | None Found |

BitLocker protection is therefore not currently enabled on `NS-W11-01`.

Encryption at rest has been identified as a future endpoint-hardening consideration pending evaluation of virtual TPM and recovery-key configuration.

## System Integrity and Maintenance

Windows image-health and protected system-file assessment tools were reviewed and practiced using:

```cmd
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
sfc /scannow
```

These commands were included as part of workstation maintenance and system-integrity practice.

No specific final DISM or SFC health result is recorded in this baseline.

Windows Update configuration, update history, advanced update options, storage-management tools, and system cleanup functionality were also reviewed during the workstation administration module.

## Domain Integration Checkpoint — September 16, 2026

This section records Phase 5 work from the supplied lab notes and retained validation screenshots. It supplements the standalone baseline rather than replacing its historical observations.

### Pre-Join Validation and DNS Configuration

Before joining the domain, the workstation was recorded as `WORKGROUP` with `CsPartOfDomain = False`.

| Setting | Pre-Join Observation | Integration State |
|---|---|---|
| IPv4 Assignment | VMware DHCP | VMware DHCP retained |
| Observed IPv4 Address | `192.168.252.128` | DHCP; no static address assigned |
| Subnet Mask | `255.255.255.0` | No change reported |
| Default Gateway | `192.168.252.2` | No change reported |
| DHCP Server | `192.168.252.254` | VMware DHCP retained |
| IPv4 DNS Server | `192.168.252.2` | `192.168.252.10` |

The notes record successful connectivity to `NS-DC01` at `192.168.252.10`, while the Northstar domain and LDAP SRV records did not resolve through VMware DNS. This demonstrated a DNS prerequisite gap despite working IP connectivity.

The workstation's IPv4 DNS server was changed to the DC's network address, `192.168.252.10`. The DC's local loopback setting `127.0.0.1` was not assigned to the workstation.

After the change, the notes record successful resolution of:

- `NS-DC01.ad.northstarsolutions.com`
- `ad.northstarsolutions.com`
- `_ldap._tcp.dc._msdcs.ad.northstarsolutions.com`
- External names such as `microsoft.com`

The pre-join findings and DNS change are written lab observations without a separate retained screenshot. They are not classified as a deliberately induced domain-join incident.

### Domain Membership and Secure Channel

The workstation was joined to `ad.northstarsolutions.com`. Validation in an administrative PowerShell session used:

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

These results are visible in the [domain-membership validation screenshot](../02-windows-server/screenshots/12-ns-w11-01-domain-membership-verification.png).

### Active Directory Computer Object

The enabled workstation computer object was moved to `Northstar > Workstations > Finance`.

Its verified Distinguished Name is:

```text
CN=NS-W11-01,OU=Finance,OU=Workstations,OU=Northstar,DC=ad,DC=northstarsolutions,DC=com
```

The [server-side computer-object verification](../02-windows-server/screenshots/13-ns-w11-01-ad-computer-object-verification.png) shows both the enabled state and OU placement. This Phase 5 placement preceded the centralized policy validation recorded below.

### Finance Domain-User Session

A separate session was validated as `NORTHSTAR\ava.chen` using:

```powershell
whoami
whoami /user
whoami /groups
$env:USERDOMAIN
$env:USERDNSDOMAIN
net localgroup Administrators
nltest /dsgetdc:ad.northstarsolutions.com
```

The [Finance session screenshot](../02-windows-server/screenshots/14-domain-user-login-and-group-membership.png) shows:

- The `NORTHSTAR\ava.chen` identity and user SID.
- `NORTHSTAR\GG-Finance-Users` in the user's Windows security token.
- `USERDOMAIN=NORTHSTAR`.
- `USERDNSDOMAIN=AD.NORTHSTARSOLUTIONS.COM`.
- Successful Domain Controller discovery.
- No individual Ava Chen entry in local Administrators and no `BUILTIN\Administrators` entry in the displayed token.

These results support the documented standard-user session and departmental membership. The domain environment variables describe the user session; they are not a separate verification of the network adapter's DNS suffix settings.

The secure-channel command at the bottom of that screenshot has no displayed result. The successful secure-channel result is documented in the separate administrative screenshot above.

### Local and Domain Identity Context

The original `NS-W11-01\labadmin` and `NS-W11-01\finance.user` entries describe local accounts from the standalone baseline. `NORTHSTAR\ava.chen` is a separate Active Directory identity, not a renamed local Finance account.

The later local Administrators output still includes `labadmin`. The integration evidence does not constitute a full reassessment of every existing local account or resource permission.

### Policy and Security Scope

The earlier Remote Desktop password-saving policy was a Local Group Policy exercise. Domain joining and OU placement were completed in Phase 5; centralized policy application was subsequently validated in Phase 6.

The Phase 5 evidence establishes domain membership, workstation trust, and a standard domain-user session. It does not establish a new Defender, firewall, BitLocker, storage, or system-integrity assessment. Those values remain the historical observations recorded above.

## Phase 6 — Centralized Group Policy Checkpoint

### Computer Policy

| Item | Recorded State |
|---|---|
| Computer | `NS-W11-01` |
| Computer OU | `Northstar > Workstations > Finance` |
| Applied GPO | `Northstar - Workstation Baseline` |
| Registry Path | `HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System` |
| Registry Value | `InactivityTimeoutSecs` |
| Verified Value | `900` seconds |

Validation in an elevated workstation session used:

```powershell
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System' -Name InactivityTimeoutSecs
gpresult /h C:\gpo-report.html /scope computer
gpresult /r /scope computer
```

The [computer-policy evidence](../02-windows-server/screenshots/15-workstation-baseline-gpo-verification.png) shows the inactivity value and the named GPO in the applied computer policies, alongside Default Domain Policy and Local Group Policy. It does not display the GPO's exact link location or a timed lock-screen test.

### Finance User Policy

| Item | Recorded State |
|---|---|
| User | `NORTHSTAR\ava.chen` |
| User OU | `Northstar > Users > Finance` |
| Applied GPO | `Northstar - Finance User Policy` |

The user session was checked separately:

```cmd
whoami
gpresult /r /scope user
```

The [Finance policy evidence](../02-windows-server/screenshots/16-finance-user-gpo-verification.png) confirms the identity, user OU, and applied policy. The exact Finance restriction setting is not specified in the supplied notes or selected screenshot.

### SIM-GPO-001 — Missing Finance User OU Link

This was a controlled simulated support incident. The Finance user remained in the correct OU, and the GPO and its configured setting still existed, but the missing OU link prevented the intended policy from applying.

After the link was restored, `gpupdate /force` completed and user-scope `gpresult` again listed the Finance GPO. The notes also record restoration of the functional restriction.

The [GPO recovery evidence](../02-windows-server/screenshots/17-simulated-gpo-link-troubleshooting.png) shows an absent applied policy followed by successful refresh and policy application. The link edit and functional restriction are recorded in the notes rather than shown directly. The final state includes the restored link.

## Phase 7 — Windows DHCP Client Checkpoint

### Configuration Evolution

Windows DHCP on `NS-DC01` replaced VMware DHCP on VMnet8. The notes record disabling VMware DHCP before activating the Windows scope, while retaining VMware NAT.

The workstation retained automatic IPv4 addressing and was changed from manually configured internal DNS to automatic DNS configuration.

| Setting | Before Migration | Initial Windows DHCP Lease | Final Reservation |
|---|---|---|---|
| IPv4 Address | `192.168.252.128` | `192.168.252.50` | `192.168.252.120` |
| Subnet Mask | `255.255.255.0` | `255.255.255.0` | `255.255.255.0` |
| Default Gateway | `192.168.252.2` | `192.168.252.2` | `192.168.252.2` |
| DHCP Server | `192.168.252.254` | `192.168.252.10` | `192.168.252.10` |
| IPv4 DNS Server | `192.168.252.10`, manual | `192.168.252.10`, DHCP | `192.168.252.10`, DHCP |
| DHCP Enabled | Yes | Yes | Yes |

The [initial Windows DHCP client evidence](../02-windows-server/screenshots/19-windows-dhcp-client-configuration-verification.png) shows `.50` after lease release and renewal. That address is a historical lease observation, not the final reservation.

### Final Client Configuration

| Setting | Recorded State |
|---|---|
| Adapter | `Ethernet0` |
| Adapter Description | Intel(R) 82574L Gigabit Network Connection |
| MAC Address | `00-0C-29-5F-31-87` |
| IPv4 Address | `192.168.252.120` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.10` |
| DNS Server | `192.168.252.10` |
| Connection-specific DNS Suffix | `ad.northstarsolutions.com` |
| DHCP Enabled | Yes |
| Address Assignment | Windows DHCP reservation |

The reservation is named `NS-W11-01` and associates `.120` with the recorded MAC address. The notes specify a DHCP-only reservation with description `Finance domain workstation`. The [DHCP Manager evidence](../02-windows-server/screenshots/22-dhcp-reservation-verification.png) shows its name and address; it does not display those additional reservation properties.

The [final client evidence](../02-windows-server/screenshots/24-simulated-dhcp-dns-option-recovery.png) shows the table's client values after the DNS-option simulation was resolved. The workstation continues to obtain its address automatically; this is not a manual static configuration.

### Scope Options and Client Validation

The notes record these options on the `Northstar Client Network` scope:

| Option | Value |
|---|---|
| 003 — Router | `192.168.252.2` |
| 006 — DNS Servers | `192.168.252.10` |
| 015 — DNS Domain Name | `ad.northstarsolutions.com` |

The scope uses an eight-day lease duration. Full scope, exclusion, and authorization details are maintained in the [server baseline](../02-windows-server/server-baseline.md).

During cutover, reservation validation, and simulated DNS-option recovery, client commands included:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

DNS cache clearing with `ipconfig /flushdns` was also used during troubleshooting. These were deliberate configuration and recovery steps, not prerequisites for every inspection.

The notes record checks of internal DNS, LDAP SRV discovery, and external resolution:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com
Resolve-DnsName _ldap._tcp.dc._msdcs.ad.northstarsolutions.com -Type SRV
Resolve-DnsName microsoft.com
nltest /dsgetdc:ad.northstarsolutions.com
```

## Phase 7 — Troubleshooting and Recovery State

### Unexpected Workstation Time Issue

The [initial investigation evidence](../02-windows-server/screenshots/20-secure-channel-time-skew-detection.png) records conflicting results:

| Check | Observed Result |
|---|---|
| `Test-ComputerSecureChannel -Verbose` | `False`; reported a broken secure channel |
| Domain Membership | `CsPartOfDomain = True` |
| `nltest /sc_verify` | `NERR_Success` |
| `nltest /sc_query` | `NERR_Success` |
| External DNS | Successful |
| Workstation Time Zone | `Pacific Standard Time` |

The notes additionally record working internal DNS and DC discovery. Incorrect workstation time was identified as a contributing condition. The results do not establish that the workstation had left the domain or that DHCP caused the discrepancy.

Time inspection and recovery used:

```powershell
Get-Date
Get-TimeZone
w32tm /query /source
w32tm /query /status
w32tm /resync
w32tm /stripchart /computer:ns-dc01.ad.northstarsolutions.com /samples:5 /dataonly
Test-ComputerSecureChannel -Verbose
```

Resynchronization followed clock correction. The [recovery evidence](../02-windows-server/screenshots/21-time-sync-secure-channel-recovery.png) shows `NS-DC01.ad.northstarsolutions.com` as the time source, successful resynchronization, offset samples around 1.7–1.9 milliseconds, and two secure-channel results returning `True`.

This was an unexpected lab incident. Recovery followed time correction, but the retained evidence does not isolate time as the sole cause. These observations describe the workstation; they do not establish the DC's exact time-zone setting.

The notes also record `Get-ADComputer` being unavailable because the workstation's Active Directory PowerShell tools were not installed. The query was performed on `NS-DC01`, where the enabled Finance workstation object was verified. The missing local command was a tooling limitation rather than evidence of an Active Directory outage.

### SIM-DHCP-001 — Incorrect DNS Option

This was a controlled simulated support incident. The first attempt used VMware's `.2` resolver, but internal names and LDAP SRV queries still resolved according to the notes. That result did not reproduce the intended fault and is preserved separately from the earlier pre-join DNS failure.

The fault was then reproduced by setting DHCP Option 006 to `1.1.1.1` and renewing the client's configuration.

| Check During Simulation | Recorded Result |
|---|---|
| Reserved Address | `192.168.252.120` retained |
| DHCP Server | `192.168.252.10` |
| Default Gateway | `192.168.252.2` |
| DNS Server | `1.1.1.1` |
| DC Reachability by IP | Successful, recorded in the notes |
| External DNS | Successful |
| Internal DC Name | Failed through the configured resolver |
| Direct DC Name Query to `.10` | Successful |

The [failure evidence](../02-windows-server/screenshots/23-simulated-dhcp-dns-option-failure.png) shows the client configuration and contrasting DNS results. The direct query used:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Server 192.168.252.10 -DnsOnly
```

Option 006 was restored to `.10`. After lease release and renewal and DNS cache clearing, the final client evidence shows restored internal DNS, successful DC name resolution and discovery, and `Test-ComputerSecureChannel -Verbose` returning `True`.

The LDAP service-name queries visible in screenshots 23–24 omit `-Type SRV`. The recovery query returns an SOA authority record, not an SRV answer. Explicit SRV validation is recorded separately in the notes.

The final baseline retains the `.120` reservation and internal DNS server `.10`; the temporary public resolver is not part of the final configuration.

## Baseline Status

**Windows 11 Workstation Baseline — Complete through Phase 7 Client Validation**

The documented standalone workstation baseline includes:

- Standardized workstation identity
- Dedicated administrator and standard-user accounts
- Least-privilege access configuration
- DHCP-based network configuration
- Local Group Policy configuration
- NTFS operating-system and Finance data volumes
- Endpoint security assessment
- BitLocker encryption assessment
- Windows system-integrity and maintenance tool practice

The later domain-integration checkpoint adds:

- Pre-join network and DNS observations
- Internal DNS configuration with VMware DHCP addressing retained
- Membership in `ad.northstarsolutions.com`
- Domain Controller discovery and secure-channel validation
- Enabled computer object in the Finance Workstations OU
- Finance domain-user authentication and departmental token membership
- Standard-user session checks

The Phase 6–7 checkpoints add:

- Computer GPO application and the 900-second inactivity value
- Finance user-policy application and simulated missing-link recovery
- Migration from VMware DHCP to Windows DHCP
- Automatic internal DNS configuration and client option validation
- DHCP reservation at `192.168.252.120`
- Unexpected time issue investigation and secure-channel recovery
- Simulated incorrect DHCP DNS-option diagnosis and recovery

`NS-W11-01` is the domain-joined Northstar Finance workstation with validated centralized policies and Windows DHCP configuration. Phase 8 — File Services and Permissions is next. Shared-resource permissions and the remaining AGDLP relationships are future work; the original local `Finance-Data (F:)` volume does not establish completion of that phase.

Related documentation: [workstation module](README.md), [server baseline](../02-windows-server/server-baseline.md), and [server screenshot guide](../02-windows-server/screenshots/README.md).
