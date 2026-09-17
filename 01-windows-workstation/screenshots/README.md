# Windows 11 Workstation — Screenshot Evidence

## Purpose

This directory contains selected visual evidence collected while configuring, administering, and troubleshooting `NS-W11-01`, the Windows 11 Pro Finance workstation used in the Northstar Solutions enterprise IT homelab.

The local images demonstrate configuration changes, administrative decisions, troubleshooting, remediation, and validation from the original workstation module. Later Phase 5–7 workstation evidence is maintained in the server screenshot directory and linked below.

Screenshots are grouped by administrative activity rather than documented individually when several images support the same task.

---

## 1. Workstation Identity and System Baseline

### Context

The Windows 11 virtual machine initially used the computer name created during its original installation.

Before preparing the system as a Northstar Solutions Finance workstation, the initial identity and system information were reviewed.

### Initial State

![Original workstation identity](01-original-system-identity.png)

The original workstation used the hostname:

`win11-demo`

The initial system information was inspected before applying the Northstar Solutions naming standard.

### Configuration Change

The workstation was renamed to:

`NS-W11-01`

Naming convention:

- `NS` — Northstar Solutions
- `W11` — Windows 11 workstation
- `01` — workstation identifier

### Validation

![Renamed workstation verification](02-workstation-renamed.png)

The hostname was verified using:

```cmd
hostname
```

Verified result:

```text
NS-W11-01
```

System information was also reviewed through Windows Settings.

![Windows system information](03-system-information.png)

The captured system information confirmed the Windows edition, version, OS build, installed RAM, processor information, and standardized device name.

### Outcome

The workstation identity was standardized and documented before additional administration was performed.

---

## 2. Local Accounts and Least-Privilege Access

### Context

The Finance workstation required separation between routine employee activity and privileged IT administration.

### Initial Account Review

![Existing local accounts](04-existing-local-accounts.png)

Existing local accounts were inspected before dedicated Northstar Solutions accounts were configured.

### Administrative Account

A dedicated local administrator account was created:

`labadmin`

![Lab administrator group membership](05-labadmin-group-membership.png)

The account was added to the local `Administrators` group.

Command-line verification was also performed:

![Lab administrator verification](06-labadmin-verification.png)

Relevant commands included:

```cmd
whoami
net localgroup administrators
```

### Standard Finance User

A separate account was configured for the simulated Finance employee:

`finance.user`

![Finance standard-user membership](07-finance-user-standard-access.png)

The account belonged to the standard `Users` group and was not granted permanent local administrator membership.

![Standard-user verification](08-standard-user-verification.png)

Command-line verification confirmed that `finance.user` was operating as the standard employee account while the administrator group remained restricted to administrative accounts.

### Decision

This configuration applies the principle of least privilege by separating daily employee activity from administrative access.

### Outcome

`NS-W11-01` was configured with distinct standard-user and administrative contexts.

---

## 3. User Account Control and Administrative Elevation

### Context

Administrative operations were tested while operating from the standard Finance user context.

Because `finance.user` does not have administrator privileges, elevated operations require authorized administrative credentials.

### UAC Elevation

![UAC administrator credential prompt](09-uac-elevation-test-1.png)

Windows User Account Control requested administrator credentials when an elevated operation was attempted.

Authorized `labadmin` credentials were supplied rather than permanently changing the Finance user's group membership.

### Elevated Administrative Context

![Elevated command prompt](09-uac-elevation-test-2.png)

After elevation, the administrative command session operated under:

```text
NS-W11-01\labadmin
```

### Decision

The standard Finance account was not promoted to administrator merely to complete an administrative task.

This preserves least-privilege access while still allowing authorized IT administration.

### Outcome

Administrative elevation functioned correctly while `finance.user` remained a standard user.

This activity is also documented in troubleshooting case `INC-001`.

---

## 4. Device and Driver Assessment

### Context

Windows Device Manager was reviewed to establish the workstation device baseline and identify driver or hardware problems.

### Initial Assessment

![Device Manager baseline](10-device-manager-baseline.png)

Device Manager showed a `Base System Device` under **Other devices** with a yellow warning indicator.

The workstation network adapter was also inspected.

![Network adapter driver information](11-network-adapter-driver.png)

The Intel(R) 82574L Gigabit Network Connection driver information was reviewed, including:

- Driver provider
- Driver date
- Driver version
- Digital signer

---

## 5. VMware Device Driver Troubleshooting

### Problem

The unidentified `Base System Device` reported:

**Code 28 — Drivers are not installed**

![Base System Device Code 28](12-base-system-device-warning.png)

### Investigation

Rather than installing an unknown driver immediately, the device Hardware IDs were inspected.

![Base System Device Hardware ID](13-base-system-device-hardware-id.png)

The primary identifier included:

```text
PCI\VEN_15AD&DEV_0740&SUBSYS_074015AD&REV_10
```

The investigation identified:

- `VEN_15AD` — VMware
- `DEV_0740` — VMware VMCI device

VMware Tools was also found to be missing from the guest operating system.

### Remediation

VMware Tools installation media was mounted to the virtual machine.

![VMware Tools installation media](14-vmware-tools-installation-media.png)

VMware Tools was installed and the workstation was restarted.

### Validation

![Device Manager after VMware Tools](15-device-manager-after-vmware-tools.png)

After restart, Device Manager was inspected again.

The previously unidentified `Base System Device` and Code 28 warning were no longer present.

### Outcome

**Resolved**

The evidence supports the troubleshooting sequence:

```text
Device warning
    ↓
Review error code
    ↓
Inspect Hardware IDs
    ↓
Identify VMware virtual hardware
    ↓
Determine VMware Tools is missing
    ↓
Install VMware Tools
    ↓
Restart
    ↓
Verify warning removed
```

This issue is also documented in `device-baseline.md` and troubleshooting case `INC-002`.

---

## 6. Performance and Process Troubleshooting

### Context

Task Manager was used to investigate elevated workstation CPU utilization and identify the processes contributing to the condition.

### Initial Process Assessment

![Task Manager process baseline](16-task-manager-processes-baseline.png)

The Processes view showed high overall CPU utilization and allowed running applications to be compared by CPU and memory consumption.

Processes were then examined more closely.

![Processes sorted by CPU](17-processes-sorted-by-cpu.png)

Microsoft Edge was identified as a significant CPU consumer.

### Resource Analysis

Task Manager Performance views were reviewed to inspect CPU and memory usage.

![CPU performance](19-cpu-performance.png)

![Memory performance](20-memory-performance.png)

The workstation had:

- 2 virtual processors
- 8 GB RAM

### Edge Process Investigation

![Expanded Microsoft Edge processes](21-edge-processes-expanded.png)

Expanding the Edge process group showed individual tabs, browser processes, subframes, and supporting processes.

The **Details** view was also used to inspect individual process IDs.

![Task Manager Details view](22-task-manager-details.png)

### Startup Review

![Startup applications](23-startup-apps-baseline.png)

Startup applications were reviewed as part of the performance assessment.

Applications were not disabled automatically simply because they appeared in the startup list; their business purpose and impact should be evaluated first.

### Remediation

A low-impact remediation was attempted by reducing the active browser workload before terminating the entire application.

During the high-usage state, Edge was responsible for a substantial portion of CPU utilization.

![High Edge CPU utilization](24-cpu-after-closing-high-usage-tabs.png)

After unnecessary browser workload was closed, overall CPU usage decreased substantially.

![CPU after Edge remediation](24-cpu-after-closing-edge.png)

The later Task Manager view showed total CPU utilization around:

`11%`

### Outcome

The investigation demonstrated that the active Edge workload was a significant contributor to the elevated CPU condition.

The troubleshooting approach prioritized identifying the responsible workload and testing a low-impact corrective action before making broader system changes.

This activity is documented as troubleshooting case `INC-003`.

---

## 7. Windows Services Administration

### Context

Windows Services were reviewed to practice administration and validation of background Windows services.

The Print Spooler service was selected for the exercise.

### Service Properties

![Print Spooler properties](25-print-spooler-service-properties.png)

The following properties were reviewed:

- Service name
- Display name
- Executable path
- Startup type
- Service status
- Dependencies

The Print Spooler executable path was shown as:

```text
C:\WINDOWS\System32\spoolsv.exe
```

### Running State

![Print Spooler running](26-print-spooler-service-running.png)

The graphical Services console showed the Print Spooler running with an Automatic startup type.

### Command-Line Validation

![Print Spooler command verification](27-print-spooler-command-verification.png)

Service state was also validated using Command Prompt and PowerShell.

Commands included:

```cmd
sc query spooler
```

and:

```powershell
Get-Service Spooler
```

The exercise captured both stopped and running service states.

### Outcome

The Print Spooler was successfully stopped, restarted, and validated using multiple Windows administration interfaces.

---

## 8. Local Group Policy Administration

### Context

Local Group Policy was configured to gain experience with Windows policy administration before introducing centralized Active Directory Group Policy later in the Northstar Solutions environment.

### Policy Configuration

The Remote Desktop Connection client policy:

**Do not allow passwords to be saved**

was configured as:

**Enabled**

![RDP password-saving policy enabled](28-rdp-password-saving-policy-enabled.png)

### Policy Application

The policy configuration was refreshed using:

```cmd
gpupdate /force
```

![Group Policy update successful](29-gpupdate-force-success.png)

Both Computer Policy and User Policy updates completed successfully.

### Policy Diagnostics

Group Policy diagnostic tools were also reviewed, including:

```cmd
gpresult /r
rsop.msc
```

![Group Policy and Resultant Set of Policy](28-local-group-policy-rdp-security.png)

### Outcome

The workstation successfully applied the configured Local Group Policy.

This provides a standalone policy-management baseline before the workstation is later integrated with centralized domain Group Policy.

---

## 9. DNS and Network Troubleshooting

### Context

Windows DNS client tools were used to practice distinguishing name resolution from other forms of network connectivity.

### DNS Resolution and Cache Administration

![DNS cache and resolution testing](29-dns-cache-flush-and-resolution.png)

Tools and commands used included:

```powershell
Resolve-DnsName microsoft.com
Get-DnsClientCache
Clear-DnsClientCache
```

and:

```cmd
ping microsoft.com
ipconfig /displaydns
ipconfig /flushdns
nslookup microsoft.com
```

### Observation

`Resolve-DnsName` and `nslookup` successfully returned IP addresses for `microsoft.com`.

At the same time, the `ping` request timed out.

### Interpretation

This demonstrated an important troubleshooting distinction:

**Successful DNS resolution does not guarantee that a target will respond to ICMP echo requests.**

A failed ping therefore does not automatically indicate DNS failure.

### Outcome

The exercise demonstrated DNS resolution, DNS client-cache management, and the importance of testing individual network functions separately.

---

## 10. Storage Administration

### Context

Additional storage was provisioned for the Finance workstation to practice Windows disk and volume administration.

### Configuration

A secondary virtual disk was added and configured using Windows Disk Management.

![Finance data volume](30-finance-data-volume.png)

The resulting volume was configured as:

| Setting | Value |
|---|---|
| Drive Letter | `F:` |
| Volume Label | `Finance-Data` |
| File System | NTFS |
| Capacity | ~10 GB |
| Status | Healthy |

The screenshot also shows successful access to the volume through File Explorer.

### Validation

A test directory was created on the `Finance-Data` volume to confirm usable read/write access.

### Outcome

The workstation gained a dedicated NTFS data volume for the simulated Finance environment.

---

## 11. Microsoft Defender Security Verification

### Context

Microsoft Defender Antivirus was reviewed as part of the workstation endpoint-security baseline.

### Evidence

![Microsoft Defender security baseline](31-defender-security-baseline.png)

Windows Security showed:

- No current threats
- Completed Quick Scan
- 0 threats found
- Security intelligence up to date

PowerShell verification also showed:

```text
AntivirusEnabled              : True
AntispywareEnabled            : True
RealTimeProtectionEnabled     : True
BehaviorMonitorEnabled        : True
```

### Outcome

Microsoft Defender Antivirus and real-time protection were verified as active during the captured assessment.

The module focused on inspecting and validating the existing Defender configuration rather than claiming deployment of a new enterprise endpoint-protection platform.

---

## 12. Windows Defender Firewall Verification

### Context

Windows Defender Firewall configuration was inspected using both the Advanced Security console and PowerShell.

### Evidence

![Windows Defender Firewall verification](32-firewall-profile-verification.png)

The captured state showed Windows Defender Firewall enabled for:

- Domain
- Private
- Public

The **Public** network profile was active at the time of the assessment.

PowerShell verification included:

```powershell
Get-NetFirewallProfile
Get-NetConnectionProfile
```

### Outcome

Windows Defender Firewall was verified as enabled across the available Windows firewall profiles.

Inbound and outbound firewall-rule administration was also reviewed through Windows Defender Firewall with Advanced Security.

---

## 13. Domain Integration and Finance User Validation

### Context

During Phase 5, `NS-W11-01` was joined to `ad.northstarsolutions.com` after its DNS client was configured to use `NS-DC01` at `192.168.252.10`. VMware DHCP addressing was retained at that checkpoint.

### Evidence

| Server-Module Screenshot | Visible Validation |
|---|---|
| [12 — Domain membership](../../02-windows-server/screenshots/12-ns-w11-01-domain-membership-verification.png) | Domain membership, DC discovery, and a `True` secure-channel result in an administrative session |
| [13 — Computer object](../../02-windows-server/screenshots/13-ns-w11-01-ad-computer-object-verification.png) | Enabled workstation object in `Northstar > Workstations > Finance` |
| [14 — Finance user session](../../02-windows-server/screenshots/14-domain-user-login-and-group-membership.png) | Ava Chen's domain identity, departmental group token, local Administrators listing, and DC discovery |

### Interpretation

`NORTHSTAR\ava.chen` is separate from the original local `finance.user` account. The secure-channel command at the bottom of screenshot 14 has no displayed result; the successful result comes from screenshot 12.

The earlier endpoint-security, hardware, and storage screenshots remain historical assessments. Domain joining did not revalidate all those controls.

### Outcome

Domain membership, computer placement, and a separate Finance domain-user session were validated.

---

## 14. Centralized Group Policy Validation and Simulated Recovery

### Context

Phase 6 introduced computer and Finance user policies, following the earlier Local Group Policy exercise in section 8. **SIM-GPO-001** was a controlled missing Finance user OU link scenario.

### Evidence

| Server-Module Screenshot | Visible Validation |
|---|---|
| [15 — Workstation policy](../../02-windows-server/screenshots/15-workstation-baseline-gpo-verification.png) | `Northstar - Workstation Baseline` applied in computer scope; `InactivityTimeoutSecs = 900` |
| [16 — Finance user policy](../../02-windows-server/screenshots/16-finance-user-gpo-verification.png) | `NORTHSTAR\ava.chen` and applied `Northstar - Finance User Policy` |
| [17 — Simulated link recovery](../../02-windows-server/screenshots/17-simulated-gpo-link-troubleshooting.png) | Earlier absent applied policy, successful refresh, and restored Finance policy result |

### Interpretation

Computer and user results were checked in separate sessions. The evidence does not specify the exact Finance restriction setting or show a timed workstation lock test.

The notes record identifying and restoring the missing Finance user OU link, followed by restoration of the functional restriction. Screenshot 17 shows the policy results, not the link edit itself.

### Outcome

Computer and user policy application were verified, and the controlled missing-link scenario was resolved.

---

## 15. Windows DHCP Client Migration and Reservation

### Context

In Phase 7, Windows DHCP on `NS-DC01` replaced VMware DHCP. The workstation retained automatic IPv4 addressing and was changed to obtain DNS automatically. VMware NAT remained at `192.168.252.2`.

### Evidence

| Server-Module Screenshot | Visible Validation |
|---|---|
| [18 — Server-side lease validation](../../02-windows-server/screenshots/18-windows-dhcp-client-lease-verification.png) | DHCP authorization, scope activation, and active workstation lease at `.50` |
| [19 — Client configuration](../../02-windows-server/screenshots/19-windows-dhcp-client-configuration-verification.png) | `.50` lease, DHCP enabled, DHCP and DNS server `.10`, gateway `.2`, and Northstar suffix |
| [22 — DHCP reservation](../../02-windows-server/screenshots/22-dhcp-reservation-verification.png) | `NS-W11-01` reservation at `192.168.252.120` in DHCP Manager |
| [24 — Final client configuration](../../02-windows-server/screenshots/24-simulated-dhcp-dns-option-recovery.png) | `.120` address, MAC `00-0C-29-5F-31-87`, DHCP enabled, and internal DNS restored after the simulation |

### Interpretation

The initial Windows lease at `.50` and later reservation at `.120` are successive build stages. The client continued to use DHCP; `.120` was not manually assigned on the workstation.

The host-side VMware DHCP change and full scope configuration are recorded in the notes and [server baseline](../../02-windows-server/server-baseline.md). The client images demonstrate the resulting configuration.

### Outcome

The workstation received addressing and internal DNS from Windows DHCP, then obtained its predictable reserved address.

---

## 16. Unexpected Time and Secure-Channel Investigation

### Context

An unexpected secure-channel discrepancy was investigated during Phase 7 validation. It was separate from the intentionally created DHCP DNS-option fault.

### Evidence

| Server-Module Screenshot | Visible Validation |
|---|---|
| [20 — Initial investigation](../../02-windows-server/screenshots/20-secure-channel-time-skew-detection.png) | Secure-channel test `False`, domain membership still `True`, successful `nltest` trust-related results, and clock inspection |
| [21 — Recovery](../../02-windows-server/screenshots/21-time-sync-secure-channel-recovery.png) | Successful time resynchronization, `NS-DC01` as source, closely synchronized offsets, and repeated secure-channel results of `True` |

### Interpretation

The notes identify incorrect workstation time as a contributing condition. The conflicting initial results are retained rather than treating one failed test as proof that the workstation left the domain.

Recovery followed clock correction and resynchronization. The evidence does not prove DHCP caused the issue or establish time as its sole cause. The visible `Pacific Standard Time` identifier describes the workstation, not the server's time-zone setting.

### Outcome

Successful time synchronization and secure-channel validation were recorded after the unexpected lab issue.

---

## 17. Simulated DHCP DNS-Option Failure and Recovery

### Context

**SIM-DHCP-001** deliberately supplied `1.1.1.1` through DHCP Option 006. An earlier attempted fault using VMware's `.2` resolver still resolved internal names according to the notes and did not reproduce the intended failure.

### Evidence

| Server-Module Screenshot | Visible Validation |
|---|---|
| [23 — Controlled failure](../../02-windows-server/screenshots/23-simulated-dhcp-dns-option-failure.png) | Reserved `.120` address, public DNS setting, successful external resolution, internal-name failures, and successful direct query to `.10` |
| [24 — Recovery](../../02-windows-server/screenshots/24-simulated-dhcp-dns-option-recovery.png) | Restored DNS `.10`, DC name resolution and discovery, and a `True` secure-channel result |

### Interpretation

The direct internal query supported isolation to the client's DNS configuration. The notes record restoring Option 006, releasing and renewing the lease, and clearing the DNS cache.

The LDAP service-name query in screenshot 24 omits `-Type SRV` and returns an SOA authority record. It is not an SRV answer; explicit SRV validation is recorded separately in the notes.

### Outcome

The simulated DNS-option fault was resolved while the workstation retained its DHCP reservation. This demonstrates why a valid lease and working external DNS alone do not establish correct Active Directory DNS configuration.

---

## Evidence Summary

The selected screenshots demonstrate practical administration of `NS-W11-01` across the following areas:

- Workstation deployment and identity standardization
- Local account administration
- Least-privilege access
- User Account Control
- Device and driver assessment
- Hardware-ID investigation
- VMware Tools remediation
- Task Manager performance troubleshooting
- Process and startup analysis
- Windows Services administration
- Local Group Policy
- DNS and network troubleshooting
- NTFS storage administration
- Microsoft Defender verification
- Windows Defender Firewall verification
- Domain membership, DC discovery, and Finance user-session validation
- Centralized computer and user Group Policy validation
- Simulated missing Finance GPO link recovery
- Windows DHCP client migration and reservation
- Unexpected time issue investigation and secure-channel recovery
- Simulated DHCP DNS-option failure isolation and recovery

The evidence is intentionally selective.

Not every command or administrative action requires its own screenshot. Detailed technical configuration is maintained in the workstation baseline documents, reusable procedures and concepts are maintained in the Knowledge Base, and troubleshooting evidence is labeled as genuine project incidents, simulated support incidents, or focused troubleshooting exercises.

See the [workstation baseline](../system-baseline.md) for configuration history and the [server screenshot guide](../../02-windows-server/screenshots/README.md) for the full Phase 5–7 evidence descriptions. The later images remain in the server directory with their existing filenames; this guide links to those originals.

## Status

**Windows 11 Workstation Screenshot Evidence — Complete through Phase 7 Client Validation**

Sections 1–12 preserve the original workstation evidence. Sections 13–17 reference the later domain, Group Policy, DHCP, and troubleshooting evidence maintained with the server module. Section numbers in this guide are activity numbers, not a new screenshot filename sequence.

Phase 8 — File Services and Permissions is next. The local Finance data-volume evidence does not establish completion of shared-resource permissions or the remaining AGDLP relationships.
