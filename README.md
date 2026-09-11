# Windows Enterprise Support Lab

Hands-on Windows Server and Active Directory administration lab built to practise common enterprise IT support, identity administration, endpoint management, and troubleshooting tasks.

## Environment

- Hypervisor: VirtualBox
- Server: Windows Server 2025
- Client: Windows 11 Enterprise
- Domain: `adlab.test`
- Domain Controller: `DC01`
- Client: `CLIENT01`
- Lab network: `10.10.10.0/24`

## Objectives

- Configure Active Directory Domain Services
- Configure DNS
- Create and manage domain users and security groups
- Join Windows clients to a domain
- Organise users and computers with Organizational Units
- Configure and troubleshoot Group Policy
- Manage SMB and NTFS permissions
- Troubleshoot common domain, DNS, policy, and authentication failures
- Document support procedures and recovery steps

## Current Progress

- [x] Created isolated VirtualBox NAT network
- [x] Installed Windows Server 2025
- [x] Configured `DC01` with a static IP
- [x] Installed AD DS and DNS
- [x] Created the `adlab.test` forest/domain
- [x] Installed and configured Windows 11 Enterprise client
- [x] Joined `CLIENT01` to the domain
- [x] Validated domain and DNS health
- [x] Created OU hierarchy
- [x] Created domain users and security groups
- [x] Moved the workstation computer object into the Workstations OU
- [x] Validated domain-user login and group membership
- [x] Practised password reset and account enable/disable tasks
- [x] Verified directory objects with PowerShell
- [x] Configured computer-targeted Group Policy
- [x] Configured user-targeted Group Policy
- [x] Reproduced and diagnosed a GPO scope failure
- [x] Validated effective policy with `gpresult`
- [x] Created the first support troubleshooting runbook
- [ ] Configure SMB and NTFS permissions
- [ ] Introduce and troubleshoot controlled DNS/domain failures
- [ ] Add further support runbooks and recovery procedures

## Documentation

- [01 - Lab Setup](docs/01-lab-setup.md)
- [02 - Active Directory Identity Management](docs/02-active-directory-identity-management.md)
- [03 - Group Policy Management and Troubleshooting](docs/03-group-policy-management-and-troubleshooting.md)

## Runbooks

- [01 - GPO Not Applying: Incorrect OU Scope](runbooks/01-gpo-not-applying-wrong-ou.md)

## Evidence

Screenshots documenting the build, validation, and troubleshooting steps are stored in the [`screenshots/`](screenshots/) directory.

## Planned Expansion

The lab will continue with SMB and NTFS permissions, account-lockout scenarios, Windows event logging, controlled DNS/domain break-fix exercises, and additional support-style runbooks.

## Scope

This is a personal lab environment. The project demonstrates hands-on administration and troubleshooting practice and is not represented as production Active Directory ownership.
