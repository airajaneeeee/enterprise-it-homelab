# SIM-GPO-001 — Missing Finance User OU Link

## Environment

- Organization: Northstar Solutions
- Domain: `ad.northstarsolutions.com`
- Domain Controller: `NS-DC01`, Windows Server 2025
- Workstation: `NS-W11-01`, Windows 11 Pro
- Platform: VMware Workstation
- Affected User: `NORTHSTAR\ava.chen`
- User OU: `Northstar > Users > Finance`
- Group Policy Object: `Northstar - Finance User Policy`
- Project Phase: 6 — Centralized Group Policy

## Scenario Classification

**Controlled simulated support incident.**

The missing Finance user OU link was intentionally used to reproduce a policy-application problem. This was not an unexpected project incident or a production outage.

## Reported Symptom

The Finance user policy stopped applying, and the intended user restriction was no longer effective according to the lab notes.

The exact restriction setting is not specified in the supplied notes or selected evidence. This report therefore identifies the affected GPO and its application state without inventing a setting or user-interface symptom.

## Initial Assessment

The affected identity and policy results were inspected on `NS-W11-01` in Ava Chen's session:

```cmd
whoami
gpresult /r /scope user
```

The affected policy was absent from the applied results. The retained troubleshooting screenshot shows an earlier applied-policy list displaying `N/A`.

## Investigation

The investigation separated user placement, GPO configuration, and the link controlling its scope.

| Check | Recorded Finding |
|---|---|
| Affected identity | Finance domain user `NORTHSTAR\ava.chen` |
| User OU | Correct Finance user OU |
| GPO existence | `Northstar - Finance User Policy` still existed |
| Configured setting | Remained enabled according to the notes |
| Applied user policies | Finance GPO absent |
| Finance user OU link | Missing |

The correct user location and existing GPO did not establish that the policy was linked to the intended scope. The missing link explained the recorded absence from the user's applied-policy results.

## Root Cause

The link for `Northstar - Finance User Policy` at the Finance user OU was missing in the controlled scenario.

The GPO itself and its enabled setting remained present. The recorded fault concerned policy scope rather than deletion of the GPO or incorrect placement of the user.

## Remediation

The GPO link was restored at `Northstar > Users > Finance`.

Policy was then refreshed on the workstation:

```cmd
gpupdate /force
```

The output reported successful computer and user policy updates. The link correction preceded the refresh; repeatedly refreshing policy alone would not have corrected the missing link.

## Verification

User-scope results were checked again in the Finance session:

```cmd
gpresult /r /scope user
```

| Validation | Result | Evidence Basis |
|---|---|---|
| Policy refresh | Computer and user updates completed successfully | Screenshot 17 |
| Applied user policy | `Northstar - Finance User Policy` listed again | Screenshot 17 |
| User context | Results for Ava Chen on `NS-W11-01` | Screenshot 17 |
| Intended restriction | Restored according to the notes | Written lab record |
| Finance user OU link | Restored according to the notes | Written lab record |

The screenshot demonstrates policy recovery. It does not display the link edit or the functional restriction's interface.

## Resolution

**Resolved — controlled scenario completed.**

The Finance user OU link was restored, the policy appeared in the user's applied results again, and the notes record successful functional validation. The final documented state retains the restored link.

## Troubleshooting Principles Applied

- Confirmed the affected user context before interpreting policy results.
- Checked OU placement, GPO existence, and the configured setting separately.
- Used the absent applied GPO to guide the scope investigation.
- Corrected the missing link before refreshing policy.
- Rechecked the applied result and the original restriction.
- Distinguished visible screenshot evidence from actions recorded in the notes.

## Lessons Learned

- A GPO existing in the domain does not prove that it applies to the intended user.
- Correct OU placement and an enabled setting still require the appropriate policy link.
- A successful refresh message should be followed by applied-policy and functional checks.
- Simulated support incidents should retain their classification when presented in reports or interviews.

## Evidence

| Screenshot | What It Supports |
|---|---|
| [16 — Finance user-policy validation](../screenshots/16-finance-user-gpo-verification.png) | Working Finance identity and applied policy before the simulated troubleshooting sequence |
| [17 — Simulated missing-link recovery](../screenshots/17-simulated-gpo-link-troubleshooting.png) | Earlier absent applied policy, successful refresh, and restored Finance policy result |

See the [server screenshot guide](../screenshots/README.md) for evidence descriptions and the [server baseline](../server-baseline.md) for the final configuration. The [Windows Server knowledge base](../../knowledge-base/windows-server-administration.md) contains reusable policy-validation concepts.
