# GPO Not Applying - Incorrect OU Scope

## Scenario

A user-targeted Group Policy Object (GPO) named `Finance-User-Baseline` was configured to restrict access to Control Panel and Windows Settings for the Finance user `alice.finance`.

The policy did not apply when it was initially linked to the Workstations OU.

## Symptoms

- `alice.finance` could still access Control Panel and Windows Settings.
- `gpupdate /force` completed successfully.
- `gpresult /r` did not list `Finance-User-Baseline` under the user's applied Group Policy Objects.

## Evidence

The GPO was initially linked to:

```text
ADLAB/Computers/Workstations
```

The target user object was located in:

```text
ADLAB/Users/Finance
```

The configured restriction was under:

```text
User Configuration
```

Relevant evidence:

- [`17-finance-policy-wrong-link.png`](../screenshots/17-finance-policy-wrong-link.png)
- [`18-finance-gpo-not-applied.png`](../screenshots/18-finance-gpo-not-applied.png)

## Root Cause

The GPO contained a **User Configuration** setting but was linked to the OU containing the computer object rather than the OU containing the target user.

Without Group Policy loopback processing, user-side policy is normally evaluated using the user's Active Directory location. Because `alice.finance` was in the Finance OU, the GPO linked only to the Workstations OU was outside the user's normal policy-processing scope.

## Resolution

The existing `Finance-User-Baseline` GPO was linked to:

```text
ADLAB/Users/Finance
```

The incorrect Workstations OU link was then removed, leaving the intended structure:

```text
ADLAB/Computers/Workstations
└── Workstation-Baseline

ADLAB/Users/Finance
└── Finance-User-Baseline
```

On `CLIENT01`, Group Policy was refreshed:

```powershell
gpupdate /force
```

The user signed out and signed back in so the user-targeted settings could be processed in a new session.

Relevant evidence:

- [`19-finance-gpo-correct-link.png`](../screenshots/19-finance-gpo-correct-link.png)

## Validation

After the correction:

- `gpresult /r` showed `Finance-User-Baseline` under the applied user Group Policy Objects.
- Control Panel access was restricted for `alice.finance` as intended.
- An HTML Group Policy Results report was generated for additional validation.

Relevant evidence:

- [`20-finance-policy-applied.png`](../screenshots/20-finance-policy-applied.png)
- [`21-gpresult-html-report.png`](../screenshots/21-gpresult-html-report.png)

## Prevention

Before linking a new GPO:

1. Identify whether the required setting is under **Computer Configuration** or **User Configuration**.
2. Identify the Active Directory object that should receive the policy.
3. Confirm the object's OU location.
4. Link the GPO at an appropriate scope.
5. Run `gpupdate /force` where appropriate.
6. Validate effective policy with `gpresult /r`, `gpresult /h`, or Resultant Set of Policy (`rsop.msc`).

## Troubleshooting Pattern

**Symptoms -> Evidence -> Cause -> Verify -> Prevent**

This exercise demonstrated that a successful `gpupdate /force` does not prove that a particular GPO is in scope. Effective-policy tools such as `gpresult` are required to confirm which GPOs were actually applied.

## Note on Loopback Processing

A user-side setting can be intentionally processed according to the computer's OU when Group Policy loopback processing is configured. Loopback processing was not enabled in this exercise; the incorrect Workstations OU link was created deliberately to demonstrate a common GPO scoping problem.
