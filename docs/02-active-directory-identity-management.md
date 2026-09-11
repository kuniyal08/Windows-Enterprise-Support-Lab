# Active Directory Identity Management

## Objective

Configure a basic organizational structure in Active Directory and
perform common user and computer administration tasks.

## Environment

- Domain: `adlab.test`
- Domain Controller: `DC01`
- Client: `CLIENT01`

## Organizational Units

ADLAB
├── Users
│   ├── IT
│   ├── Finance
│   └── Operations
├── Computers
│   └── Workstations
├── Servers
└── Groups

## Security Groups

- `GG_IT_Support`
- `GG_Finance`
- `GG_Operations`

## Test Accounts

- `it.support`
- `alice.finance`
- `bob.operations`

## Tasks Performed

- Created organizational units
- Created domain users
- Created security groups
- Added users to security groups
- Moved a domain computer into the Workstations OU
- Logged into Windows 11 using a domain account
- Reset a domain user's password
- Disabled and re-enabled a domain account
- Verified directory objects using PowerShell

## Validation

`CLIENT01` successfully authenticated `alice.finance` against the
`adlab.test` domain and returned the expected domain and group
membership information.

## Evidence

See the `screenshots/` directory.
