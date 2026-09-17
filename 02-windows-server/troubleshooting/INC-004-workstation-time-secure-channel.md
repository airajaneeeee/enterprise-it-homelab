# INC-004 — Workstation Time and Secure-Channel Validation Issue

## Environment

- Organization: Northstar Solutions
- Domain: `ad.northstarsolutions.com`
- Domain Controller / DNS / DHCP Server: `NS-DC01`, Windows Server 2025
- Server IPv4 Address: `192.168.252.10`
- Workstation: `NS-W11-01`, Windows 11 Pro
- Platform: VMware Workstation, VMnet8 NAT
- Project Phase: 7 — Windows DHCP, post-migration validation

## Incident Classification

**Genuine unexpected lab incident.**

The secure-channel test failure was discovered during validation after DHCP migration. It was not intentionally introduced and is separate from `SIM-DHCP-001`. Its timing does not establish that DHCP caused the issue.

## Reported Symptom

In an administrative PowerShell session on the workstation, this check returned `False`:

```powershell
Test-ComputerSecureChannel -Verbose
```

The verbose output stated:

```text
The secure channel between the local computer and the domain ad.northstarsolutions.com is broken.
```

This was a validation finding. The record does not describe a separate user-reported sign-in outage or establish an outage duration.

## Initial Assessment

The workstation's membership and related domain checks were inspected before selecting a corrective action.

```powershell
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain
nltest /sc_verify:ad.northstarsolutions.com
nltest /sc_query:ad.northstarsolutions.com
```

| Check | Recorded Result | Evidence Basis |
|---|---|---|
| Computer identity | `NS-W11-01` | Screenshot 20 |
| Domain | `ad.northstarsolutions.com` | Screenshot 20 |
| Domain membership | `CsPartOfDomain = True` | Screenshot 20 |
| PowerShell secure-channel test | `False` | Screenshot 20 |
| `nltest /sc_verify` | `NERR_Success` | Screenshot 20 |
| `nltest /sc_query` | `NERR_Success` | Screenshot 20 |
| External DNS resolution | Successful | Screenshot 20 |
| Internal DC-name resolution | Successful | Written lab record |
| LDAP SRV resolution | Successful | Written lab record |
| Domain Controller discovery | Successful | Written lab record |

The results did not consistently indicate failure. Domain membership, DNS, and discovery were retained as separate observations rather than treating the failed PowerShell test as proof that the workstation had left the domain.

## Investigation

### Workstation Clock and Time Source

The notes record that the workstation clock was significantly incorrect. Time inspection included:

```powershell
Get-Date
Get-TimeZone
w32tm /query /status
w32tm /query /source
```

The workstation reported `NS-DC01.ad.northstarsolutions.com` as its time source. Screenshot 20 shows the clock inspection and time-zone ID `Pacific Standard Time`, but it does not directly measure the initial offset from the DC.

The exact initial offset and the reason the clock became incorrect were not established in the retained record. No cause such as VMware synchronization, a suspended VM, or an incorrect server time source is inferred.

### Active Directory Tooling Observation

An attempted `Get-ADComputer` query on the workstation was not recognized because the Active Directory PowerShell tools were not installed there, according to the notes.

The directory query was performed on `NS-DC01`, where the tools were available. Screenshot 18 shows the workstation object enabled in the Finance Workstations OU. This was a local tooling limitation, separate from the secure-channel discrepancy.

## Root Cause Assessment

Incorrect workstation time was identified as a contributing condition, and secure-channel validation succeeded after clock correction and resynchronization.

The evidence supports that recovery sequence. It does not isolate time as the sole cause of the failed test, explain why the clock became incorrect, or prove that DHCP migration caused the discrepancy. The successful `nltest` results remain part of the incident record.

## Remediation

The workstation clock was corrected according to the notes. The exact clock-adjustment method was not retained.

The workstation then requested time synchronization:

```cmd
w32tm /resync
```

The command completed successfully. The recorded recovery actions were clock correction and resynchronization; the record does not document a trust reset or domain rejoin as a remediation step.

## Verification

The workstation's time source, synchronization status, offset, and secure-channel result were checked again:

```powershell
w32tm /query /source
w32tm /query /status
w32tm /stripchart /computer:ns-dc01.ad.northstarsolutions.com /samples:5 /dataonly
Test-ComputerSecureChannel -Verbose
Get-Date
Get-TimeZone
```

Screenshot 21 shows:

| Validation | Visible Result |
|---|---|
| Resynchronization | Completed successfully |
| Time source | `NS-DC01.ad.northstarsolutions.com` |
| Source IP in time status | `192.168.252.10` |
| Strip-chart offset samples | Approximately `+0.0017` to `+0.0019` seconds |
| Secure-channel validation | Two results of `True`; reported in good condition |
| Workstation time-zone ID | `Pacific Standard Time` |

The five displayed samples are approximately 1.7–1.9 milliseconds. The source notes approximate the recovered offset as 0.1 seconds; this report uses the values visible in the retained screenshot.

These samples demonstrate close synchronization at the time of the check, not a long-term time-accuracy measurement. The time-zone result describes the workstation and does not establish the server's time-zone configuration.

## Resolution

**Resolved at the recorded validation checkpoint.**

After workstation clock correction and successful resynchronization, repeated secure-channel checks returned `True`. The workstation remained domain joined. The underlying reason for the clock discrepancy was not established.

## Troubleshooting Principles Applied

- Recorded the original command result and administrative context.
- Checked domain membership, DNS, discovery, and trust-related results separately.
- Preserved conflicting results instead of replacing them with a single diagnosis.
- Investigated the clock and time source before selecting recovery actions.
- Distinguished a missing local management command from a directory-service failure.
- Repeated the failed validation after remediation.
- Separated the observed recovery from an unproven causal explanation.

## Lessons Learned

- Successful DNS and DC discovery do not establish that every domain-related check will succeed.
- A failed validation command should be investigated alongside other evidence.
- A selected time source and an observed offset provide different information.
- Recovery can be verified even when the initiating cause remains uncertain.
- An unexpected issue found during a change should not automatically be attributed to that change.

## Evidence

| Screenshot | What It Supports |
|---|---|
| [18 — Server-side validation](../screenshots/18-windows-dhcp-client-lease-verification.png) | Enabled workstation computer object in the Finance Workstations OU |
| [20 — Initial investigation](../screenshots/20-secure-channel-time-skew-detection.png) | Failed PowerShell test, retained domain membership, successful `nltest` checks, external DNS, and clock inspection |
| [21 — Recovery](../screenshots/21-time-sync-secure-channel-recovery.png) | Successful resynchronization, DC time source, measured offsets, and repeated secure-channel success |

See the [server screenshot guide](../screenshots/README.md) for evidence descriptions and the [workstation baseline](../../01-windows-workstation/system-baseline.md) for configuration history. The separate [SIM-DHCP-001 report](SIM-DHCP-001-incorrect-dns-option.md) documents the deliberately introduced DNS-option fault.
