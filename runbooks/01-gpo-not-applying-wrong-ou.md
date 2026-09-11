# GPO Not Applying - Incorrect OU Scope

## Scenario

A user-targeted Group Policy Object named
`Finance-User-Baseline` was configured to restrict access to
Control Panel and Windows Settings.

The policy did not apply to `alice.finance`.

## Symptoms

- `alice.finance` could still access Control Panel.
- `gpupdate /force` completed successfully.
- `gpresult /r` did not list `Finance-User-Baseline`
  under the user's applied GPOs.

## Evidence

The GPO was linked to:

`ADLAB/Computers/Workstations`

The user object was located in:

`ADLAB/Users/Finance`

The configured policy was under:

`User Configuration`.

## Root Cause

The GPO was linked to the OU containing the computer object
rather than the OU containing the target user.

Without Group Policy loopback processing, user policy is
normally evaluated through the user's Active Directory OU path.

## Resolution

Linked `Finance-User-Baseline` to:

`ADLAB/Users/Finance`

Removed the incorrect link from the Workstations OU.

Ran:

```powershell
gpupdate /force
