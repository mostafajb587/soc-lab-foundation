# Active Directory

## Objective

Deploy and configure a centralized Active Directory environment for the lab, providing centralized identity, authentication, authorization, and administrative organization.

## Domain

The lab uses the following Active Directory domain:

```text
corp.local
```

The corresponding NetBIOS domain name is:

```text
CORP
```

Active Directory-integrated DNS is used to support domain discovery, name resolution, and authentication.

## Domain Controller

The Domain Controller provides the core identity services for the environment:

* Active Directory Domain Services (AD DS)
* DNS
* Domain authentication
* Kerberos-based authentication
* Centralized identity management

The Domain Controller is located in the **SERVERS** network:

```text
Network: 10.10.20.0/24
DNS / Domain Controller: 10.10.20.10
```

## Organizational Unit Structure

The Active Directory structure was organized using dedicated OUs:

```text
corp.local
│
├── Corporate Users
│
├── Corporate Computers
│
├── Corporate Servers
│
├── Security Groups
│
└── IT Administration
```

The OUs separate users, endpoints, servers, security groups, and administrative identities.

This structure provides a foundation for applying targeted Group Policies and managing the environment in a controlled way.

## Users

The lab contains separate normal-user and administrative identities.

### Standard User

```text
Mostafa Jbili
```

The normal user account is used for regular domain activity and endpoint authentication.

### Administrative User

```text
itadmin
```

The administrative account is intended for IT administration rather than normal daily activity.

This follows the principle of separating standard user activity from administrative operations.

## Security Groups

The following security groups were created:

```text
SOC-Analysts
IT-Admins
```

Membership is based on role rather than individual permissions.

Example:

```text
Mostafa Jbili
    ↓
SOC-Analysts

itadmin
    ↓
IT-Admins
```

This provides a foundation for role-based access control and future security management.

## Domain Computer Organization

The Windows endpoint was joined to the `corp.local` domain and organized under:

```text
Corporate Computers
└── SOC-WIN01
```

The endpoint was initially created using the default Windows computer name and was later renamed to:

```text
SOC-WIN01
```

The computer object was then moved from the default `Computers` container into the dedicated `Corporate Computers` OU.

## Domain Join

`SOC-WIN01` was successfully joined to:

```text
corp.local
```

The endpoint uses the Domain Controller as its DNS server:

```text
DNS: 10.10.20.10
```

This allows Windows to discover domain services correctly.

## Authentication

Domain authentication was validated using the standard user account:

```text
CORP\Mostafa
```

The following command was used on `SOC-WIN01`:

```cmd
whoami
```

The result confirmed:

```text
corp\mostafa
```

This verified that the endpoint was authenticating against the domain rather than using a local account.

## Domain Controller Discovery

Domain Controller discovery was validated using:

```cmd
nltest /dsgetdc:corp.local
```

The command completed successfully and returned the trusted Domain Controller.

## Secure Channel Validation

The secure relationship between the endpoint and the domain was validated using:

```cmd
nltest /sc_verify:corp.local
```

The result confirmed:

```text
NERR_Success
```

for both the Domain Controller connection status and trust verification.

This confirms that the secure channel between `SOC-WIN01` and the domain is functioning correctly.

## DNS Validation

Domain name resolution was validated using:

```cmd
nslookup corp.local
```

The result resolved:

```text
corp.local
→ 10.10.20.10
```

This confirms that the endpoint can resolve the Active Directory domain through the configured DNS server.

## Kerberos

Kerberos is the primary authentication protocol used by the Active Directory environment.

Kerberos auditing was enabled as part of the Windows security baseline to provide authentication-related telemetry for future SOC monitoring projects.

Kerberos monitoring and detection engineering are intentionally outside the scope of this project.

## Administrative Separation

Administrative and standard user identities are separated:

```text
Standard User
    ↓
Mostafa Jbili
    ↓
SOC-Analysts

Administrative User
    ↓
itadmin
    ↓
IT-Admins
```

The standard user account is intended for normal activity, while the administrative account is reserved for administrative tasks.

This reduces the need to perform routine endpoint activity using privileged credentials.

## Group Policy Integration

The domain structure provides the foundation for centralized Group Policy management.

The `Default Domain Policy` was used to apply the initial security baseline to domain computers.

The resulting policy configuration includes:

* Password security
* Account lockout
* Authentication auditing
* Account management auditing
* Process creation auditing
* Kerberos auditing
* System integrity auditing

The detailed configuration is documented in:

```text
docs/security-baseline.md
```

## Security Considerations

The Active Directory design follows several security principles:

* Separate standard and administrative accounts.
* Organize users and computers into dedicated OUs.
* Use role-based security groups.
* Keep domain infrastructure inside the Servers network.
* Use centralized DNS for domain services.
* Apply centralized security policies through Group Policy.
* Avoid using highly privileged accounts for normal user activity.

## Validation Summary

The following Active Directory functions were successfully validated:

| Validation                  | Result |
| --------------------------- | ------ |
| DNS resolution              | Passed |
| Domain Controller discovery | Passed |
| Domain Join                 | Passed |
| Domain authentication       | Passed |
| `whoami` domain identity    | Passed |
| Secure channel verification | Passed |
| Computer OU placement       | Passed |
| Security group membership   | Passed |
| Group Policy application    | Passed |

## Result

The lab now contains a functional centralized identity environment based on Active Directory.

The environment provides:

* Centralized authentication
* DNS-based domain discovery
* Organizational Units
* Standard and administrative identities
* Role-based security groups
* Domain-joined Windows endpoint
* Centralized Group Policy
* Kerberos-based authentication foundation

This Active Directory foundation will support future SOC monitoring, identity investigation, and detection projects without implementing those components in the current project.
