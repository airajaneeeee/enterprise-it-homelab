# SIM-DHCP-001 — Incorrect DHCP DNS Option

## Environment

- Organization: Northstar Solutions
- Domain: `ad.northstarsolutions.com`
- Domain Controller / DNS / DHCP Server: `NS-DC01`, Windows Server 2025
- Server IPv4 Address: `192.168.252.10`
- Workstation: `NS-W11-01`, Windows 11 Pro
- Platform: VMware Workstation, VMnet8 NAT
- Subnet: `192.168.252.0/24`
- Default Gateway: `192.168.252.2`
- DHCP Scope: `Northstar Client Network`
- Workstation Reservation: `192.168.252.120`
- Client MAC Address: `00-0C-29-5F-31-87`
- Project Phase: 7 — Windows DHCP

## Scenario Classification

**Controlled simulated support incident.**

DHCP Option 006 was deliberately changed to reproduce a domain workstation receiving a valid lease with an incorrect DNS server. This was separate from the unexpected time and secure-channel issue investigated during Phase 7.

## Reported Symptom

After the controlled DNS-option change, the workstation could resolve external names but failed to resolve internal Northstar names. It retained its reserved IPv4 address and could reach the DC by IP according to the lab notes.

## Initial Assessment

The workstation was configured to obtain IPv4 and DNS settings automatically. Before the simulation, Windows DHCP supplied internal DNS at `192.168.252.10`.

### First Attempt — VMware Resolver

Option 006 was first changed to `192.168.252.2`. The expectation was that the VMware resolver would fail to resolve Northstar's internal records.

The notes instead record successful internal DC-name and LDAP service-name resolution, including a successful direct DC-name query after cache clearing:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Server 192.168.252.2 -DnsOnly
```

This attempt did not reproduce the intended failure. The result is retained without assigning an unverified caching or forwarding explanation. It is also separate from the earlier Phase 5 pre-join DNS failure.

### Revised Fault — Public Resolver

Option 006 was then changed to `1.1.1.1`. The client renewed its configuration and cleared its DNS cache.

The failure screenshot's `ipconfig /all` output shows:

| Setting | Observed Value |
|---|---|
| IPv4 Address | `192.168.252.120` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.252.2` |
| DHCP Server | `192.168.252.10` |
| DNS Server | `1.1.1.1` |
| Connection-specific DNS Suffix | `ad.northstarsolutions.com` |
| DHCP Enabled | Yes |

## Investigation

Testing separated address allocation, IP connectivity, external DNS, and internal resolution.

| Check | Result | Evidence Basis |
|---|---|---|
| Reserved client address | `.120` retained | Screenshot 23 |
| DC reachability by IP | Successful | Written lab record |
| `microsoft.com` resolution | Successful | Screenshot 23 |
| Internal DC name through configured resolver | DNS name does not exist | Screenshot 23 |
| Explicit LDAP SRV query | DNS name error | Written lab record |
| DC-name query explicitly targeting `.10` | Returned `192.168.252.10` | Screenshot 23 |

The notes record these connectivity and name-resolution checks:

```powershell
Test-Connection 192.168.252.10 -Count 2
Resolve-DnsName microsoft.com
Resolve-DnsName ns-dc01.ad.northstarsolutions.com
Resolve-DnsName _ldap._tcp.dc._msdcs.ad.northstarsolutions.com -Type SRV
```

The internal DNS server was then queried directly:

```powershell
Resolve-DnsName ns-dc01.ad.northstarsolutions.com -Server 192.168.252.10 -DnsOnly
```

The direct query returned the expected DC address. Combined with the public resolver shown in the client configuration, this isolated the recorded fault to the DNS server supplied to the workstation.

The visible LDAP service-name query in screenshot 23 omits `-Type SRV`. The explicit SRV failure is recorded in the notes rather than demonstrated by that particular command.

## Root Cause

DHCP Option 006 had been intentionally configured to supply `1.1.1.1` instead of Northstar's internal DNS server, `192.168.252.10`.

The client retained a valid lease and gateway, but the supplied resolver did not resolve the internal records tested in this scenario. The successful direct query showed that the internal DNS server could answer the DC-name request.

## Remediation

Option 006 was restored to `192.168.252.10` on the Windows DHCP scope.

The workstation configuration was refreshed using:

```cmd
ipconfig /release
ipconfig /renew
ipconfig /flushdns
ipconfig /all
```

Lease renewal obtained the corrected configuration, and cache clearing removed existing resolver-cache entries before retesting. Cache clearing alone would not have corrected the scope option.

## Verification

The recovery screenshot and written notes record:

| Validation | Result | Evidence Basis |
|---|---|---|
| Reserved address | `192.168.252.120` | Screenshot 24 |
| DHCP enabled | Yes | Screenshot 24 |
| DHCP server | `192.168.252.10` | Screenshot 24 |
| DNS server | `192.168.252.10` | Screenshot 24 |
| Gateway | `192.168.252.2` | Screenshot 24 |
| Connection-specific suffix | `ad.northstarsolutions.com` | Screenshot 24 |
| Ordinary DC-name resolution | Returned `.10` | Screenshot 24 |
| Direct DC-name query to internal DNS | Returned `.10` | Screenshot 24 |
| Domain Controller discovery | Located `NS-DC01` successfully | Screenshot 24 |
| Secure-channel test | `True`; reported in good condition | Screenshot 24 |
| Explicit LDAP SRV resolution | Restored | Written lab record |

The visible domain checks used:

```powershell
nltest /dsgetdc:ad.northstarsolutions.com
Test-ComputerSecureChannel -Verbose
```

The LDAP service-name query visible in screenshot 24 omits `-Type SRV` and returns an SOA record in the Authority section. That output is not an SRV answer. Successful explicit SRV validation is recorded separately in the notes.

## Resolution

**Resolved — controlled scenario completed.**

The internal DNS option was restored, and the client again resolved the DC name and completed DC discovery and secure-channel validation. The `.120` reservation remained in use. The temporary public DNS setting is not part of the final baseline.

## Troubleshooting Principles Applied

- Inspected actual client configuration before interpreting DNS failures.
- Preserved the first test result when it did not reproduce the expected fault.
- Tested IP connectivity, external resolution, and internal resolution separately.
- Compared the configured resolver with an explicit internal DNS query.
- Corrected the DHCP option before renewing client configuration.
- Rechecked the original failure and domain validation after recovery.
- Distinguished visible record types from SRV tests reported in the notes.

## Lessons Learned

- A valid DHCP lease does not prove that the client received the correct DNS server.
- Successful external resolution does not establish internal Active Directory name resolution.
- DHCP Option 006 supplies a client DNS setting; it is separate from DNS Server forwarding configuration.
- An unsuccessful attempt to reproduce a fault is evidence that should remain in the record.
- A DNS response should be interpreted according to its record type and the test performed.

## Evidence

| Screenshot | What It Supports |
|---|---|
| [23 — Simulated DNS-option failure](../screenshots/23-simulated-dhcp-dns-option-failure.png) | Public DNS client setting, retained lease, successful external resolution, internal-name failures, and successful direct internal query |
| [24 — Simulated DNS-option recovery](../screenshots/24-simulated-dhcp-dns-option-recovery.png) | Restored internal DNS setting, retained reservation, DC-name resolution, discovery, and secure-channel success |

See the [server screenshot guide](../screenshots/README.md) for evidence descriptions, the [server baseline](../server-baseline.md) for scope configuration, and the [workstation baseline](../../01-windows-workstation/system-baseline.md) for client configuration history.
