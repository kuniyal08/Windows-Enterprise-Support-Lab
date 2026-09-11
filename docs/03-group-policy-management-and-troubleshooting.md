# Group Policy Management and Troubleshooting

## Objective

Configure and validate Group Policy in the `adlab.test` domain, then reproduce and troubleshoot a user-policy scoping failure.

This phase demonstrates both successful policy deployment and a controlled break/fix scenario using Group Policy Management, `gpupdate`, `gpresult`, and Resultant Set of Policy concepts.

## Environment

- Domain: `adlab.test`
- Domain Controller: `DC01`
- Domain Client: `CLIENT01`
- Workstation OU: `ADLAB/Computers/Workstations`
- Finance user OU: `ADLAB/Users/Finance`
- Test user: `alice.finance`

## 1. Workstation Baseline GPO

A computer-targeted GPO named `Workstation-Baseline` was created and linked to the Workstations OU:

```text
ADLAB
└── Computers
    └── Workstations
        └── Workstation-Baseline
```

The GPO configured workstation security settings under **Computer Configuration**, including:

- Machine inactivity limit: 900 seconds
- Interactive logon message title: `ADLAB Authorized Use`
- Interactive logon message text for authorised-use notification

Evidence:

- [`13-workstation-baseline-gpo.png`](../screenshots/13-workstation-baseline-gpo.png)
- [`14-workstation-baseline-settings.png`](../screenshots/14-workstation-baseline-settings.png)

## 2. Computer Policy Validation

On `CLIENT01`, Group Policy was refreshed with:

```powershell
gpupdate /force
```

The applied computer policy was then checked with:

```powershell
gpresult /r
```

`Workstation-Baseline` appeared under the applied Group Policy Objects for the computer context.

After restart, the configured ADLAB logon message was displayed, providing visible confirmation that the workstation policy had been processed.

Evidence:

- [`15-gpresult-workstation-baseline.png`](../screenshots/15-gpresult-workstation-baseline.png)
- [`16-gpo-logon-message.png`](../screenshots/16-gpo-logon-message.png)

## 3. Deliberate User-Policy Scoping Failure

A second GPO named `Finance-User-Baseline` was created. It contained a **User Configuration** policy that prohibited access to Control Panel and Windows Settings.

For the troubleshooting exercise, the GPO was intentionally linked to the Workstations OU rather than the Finance user OU.

Initial configuration:

```text
ADLAB
└── Computers
    └── Workstations
        ├── CLIENT01
        ├── Workstation-Baseline
        └── Finance-User-Baseline   <- intentionally incorrect link
```

The target user remained located at:

```text
ADLAB
└── Users
    └── Finance
        └── alice.finance
```

Evidence:

- [`17-finance-policy-wrong-link.png`](../screenshots/17-finance-policy-wrong-link.png)

## 4. Failure Validation

`alice.finance` logged into `CLIENT01` and Group Policy was refreshed.

The following command was used to inspect effective policy:

```powershell
gpresult /r
```

`Finance-User-Baseline` was not listed as an applied user GPO, and the configured Control Panel restriction was not enforced.

This showed that `gpupdate /force` completing successfully did not mean that every configured GPO was applicable to the current user.

Evidence:

- [`18-finance-gpo-not-applied.png`](../screenshots/18-finance-gpo-not-applied.png)

## 5. Root Cause

The setting being tested was a **User Configuration** policy, but the GPO was linked only to the OU containing the computer object.

Without Group Policy loopback processing, user-side policy is normally evaluated according to the user's Active Directory OU path.

In this lab:

```text
Computer object:
ADLAB/Computers/Workstations/CLIENT01

User object:
ADLAB/Users/Finance/alice.finance
```

Because `alice.finance` was not within the scope of the Workstations OU link, the user-targeted GPO did not apply.

## 6. Corrective Action

The existing `Finance-User-Baseline` GPO was linked to the Finance OU:

```text
ADLAB
└── Users
    └── Finance
        └── Finance-User-Baseline
```

The intentionally incorrect Workstations OU link was removed.

Evidence:

- [`19-finance-gpo-correct-link.png`](../screenshots/19-finance-gpo-correct-link.png)

On `CLIENT01`, policy was refreshed again:

```powershell
gpupdate /force
```

The user then signed out and signed back in so the user-targeted settings could be processed in a new session.

## 7. Corrected Policy Validation

After the GPO was linked to the Finance OU:

- `Finance-User-Baseline` appeared in the applied user GPOs.
- Control Panel and Windows Settings access was restricted for `alice.finance` as intended.

Evidence:

- [`20-finance-policy-applied.png`](../screenshots/20-finance-policy-applied.png)

## 8. Group Policy Results Reporting

An HTML Group Policy Results report was generated on `CLIENT01` using:

```powershell
New-Item -ItemType Directory -Path C:\Temp -Force
gpresult /h C:\Temp\gpresult.html
```

The report provided a more detailed view of effective Group Policy than the concise `/r` output.

Evidence:

- [`21-gpresult-html-report.png`](../screenshots/21-gpresult-html-report.png)

Resultant Set of Policy can also be inspected with:

```powershell
rsop.msc
```

## 9. Key Concepts Demonstrated

- Creating and linking Group Policy Objects
- Computer Configuration versus User Configuration
- OU-based GPO scope
- Group Policy refresh with `gpupdate`
- Effective-policy validation with `gpresult`
- HTML Group Policy Results reporting
- Controlled GPO failure reproduction
- Root-cause analysis of incorrect OU scope
- Corrective linking and validation
- Awareness of Group Policy loopback processing as a separate, intentional mechanism

## Troubleshooting Lesson

A successful Group Policy refresh does not prove that a specific GPO is in scope.

A useful troubleshooting sequence is:

```text
1. Confirm whether the setting is User or Computer Configuration
2. Confirm the target AD object's OU
3. Inspect where the GPO is linked
4. Refresh policy
5. Use gpresult to verify effective policy
6. Correct the scope or filtering issue
7. Re-validate the intended setting
```

For the detailed break/fix procedure, see:

[Runbook: GPO Not Applying - Incorrect OU Scope](../runbooks/01-gpo-not-applying-wrong-ou.md)
