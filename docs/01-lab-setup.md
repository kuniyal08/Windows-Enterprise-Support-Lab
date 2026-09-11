# Windows Enterprise Support Lab Setup

## Objective

Build a small Windows enterprise lab for hands-on practice with Windows Server, Active Directory Domain Services (AD DS), DNS, domain-joined clients, identity administration, Group Policy, permissions, and common support troubleshooting.

This lab is intentionally small enough to run on a personal workstation while still reproducing the core relationships found in a Windows domain environment.

## Environment

| Component | Configuration |
|---|---|
| Hypervisor | VirtualBox |
| Virtual network | `ADLAB` NAT Network |
| Network | `10.10.10.0/24` |
| Domain controller | `DC01` |
| Server OS | Windows Server 2025 |
| Domain controller IP | `10.10.10.10` |
| Active Directory domain | `adlab.test` |
| Client | `CLIENT01` |
| Client OS | Windows 11 Enterprise |
| Client IP | `10.10.10.20` |
| Client DNS server | `10.10.10.10` |

## Lab Architecture

```text
Fedora Host
└── VirtualBox
    └── NAT Network: ADLAB (10.10.10.0/24)
        │
        ├── DC01
        │   ├── Windows Server 2025
        │   ├── 10.10.10.10
        │   ├── Active Directory Domain Services
        │   └── DNS
        │
        └── CLIENT01
            ├── Windows 11 Enterprise
            ├── 10.10.10.20
            ├── DNS: 10.10.10.10
            └── Domain member: adlab.test
```

## 1. Virtual Network

A dedicated VirtualBox NAT Network named `ADLAB` was created with the IPv4 prefix `10.10.10.0/24`.

Using a NAT Network allows the two virtual machines to communicate with each other while retaining outbound network access for updates and package retrieval.

Evidence: [`01-virtualbox-adlab-network.png`](../screenshots/01-virtualbox-adlab-network.png)

## 2. Domain Controller Installation

A Windows Server 2025 virtual machine was created and named `DC01`.

The server was updated, renamed, and configured with the static IPv4 address:

```text
IP address:       10.10.10.10
Subnet mask:      255.255.255.0
Default gateway:  10.10.10.1
```

Evidence: [`02-dc01-server-manager.png`](../screenshots/02-dc01-server-manager.png)

## 3. Active Directory Domain Services and DNS

The Active Directory Domain Services role was installed on `DC01` and the server was promoted to a domain controller for a new forest.

The lab domain is:

```text
adlab.test
```

DNS was installed with AD DS. After promotion, the domain controller uses its own DNS service at `10.10.10.10` for Active Directory name resolution.

Basic validation included checking the domain, forest, domain controller, and DNS records with commands such as:

```powershell
Get-ADDomain
Get-ADForest
Get-ADDomainController
Resolve-DnsName adlab.test
Resolve-DnsName DC01.adlab.test
```

Evidence:

- [`03-adds-installed.png`](../screenshots/03-adds-installed.png)
- [`04-active-directory-users-computers.png`](../screenshots/04-active-directory-users-computers.png)

## 4. Windows 11 Domain Client

A Windows 11 Enterprise virtual machine was created and named `CLIENT01`.

Its lab network settings are:

```text
IP address:       10.10.10.20
Subnet mask:      255.255.255.0
Default gateway:  10.10.10.1
Preferred DNS:    10.10.10.10
```

The DNS setting is important because Active Directory clients must be able to resolve the domain controller and AD-specific DNS records through the domain DNS service.

Before joining the domain, DNS connectivity was checked using commands such as:

```powershell
ipconfig /all
nslookup adlab.test
nslookup DC01.adlab.test
Test-NetConnection DC01.adlab.test -Port 53
```

`CLIENT01` was then joined to the `adlab.test` domain and restarted.

Evidence: [`05-client-domain-membership-verified.png`](../screenshots/05-client-domain-membership-verified.png)

## 5. Domain Health Validation

After the domain controller and client were operational, the environment was validated from `DC01`.

Checks included:

```powershell
Get-ADDomain
Get-ADForest
Get-ADDomainController
dcdiag /q
Resolve-DnsName adlab.test
Resolve-DnsName DC01.adlab.test
Get-ADComputer CLIENT01
```

The domain controller responded correctly, DNS records resolved, and `CLIENT01` appeared as an Active Directory computer object.

Evidence: [`06-domain-health-check.png`](../screenshots/06-domain-health-check.png)

## 6. Baseline Snapshots

After the domain controller and client were working correctly, VirtualBox snapshots were created before continuing with identity administration and troubleshooting exercises.

This provides a known-good recovery point before introducing Group Policy, permissions, and deliberate failure scenarios.

## Key Lessons

- Active Directory is tightly dependent on DNS; a client with incorrect DNS settings can appear to have an authentication or domain-join problem even when AD DS itself is healthy.
- A domain controller should have a stable IP address because domain clients depend on it for directory and DNS services.
- Validating name resolution and network connectivity before attempting a domain join reduces unnecessary troubleshooting.
- A clean baseline snapshot makes controlled break/fix exercises repeatable and safe.

## Next Stage

The next stage of the project covers Active Directory identity administration, including:

- Organizational Units (OUs)
- Domain users
- Security groups
- Computer-object placement
- Domain-user authentication
- Password reset
- Account disable/enable operations
- PowerShell inventory and validation

See: [Active Directory Identity Management](02-active-directory-identity-management.md)
