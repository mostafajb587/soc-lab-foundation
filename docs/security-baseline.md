# Security Baseline

## Objective

Establish a basic Windows security baseline for the enterprise lab endpoint and generate security-relevant Windows telemetry for future monitoring projects.

The baseline is applied through the domain-level **Default Domain Policy** and validated on `SOC-WIN01`.

## Scope

This baseline currently covers:

* Password policy
* Account lockout policy
* Windows security auditing
* Process creation auditing
* Kerberos authentication auditing
* Basic domain endpoint security configuration

Advanced endpoint monitoring, SIEM integration, Sysmon, IDS/IPS, detection engineering, incident response automation, and SOAR are outside the scope of this project and will be implemented in later projects.

## Password Policy

The following domain password requirements were configured:

| Policy                   |         Value |
| ------------------------ | ------------: |
| Minimum password length  | 12 characters |
| Password complexity      |       Enabled |
| Enforce password history |  24 passwords |
| Maximum password age     |       60 days |
| Minimum password age     |         1 day |

### Security Rationale

These settings provide a reasonable baseline for the lab by increasing password strength, preventing immediate password reuse, and limiting long-term password exposure.

## Account Lockout Policy

The following account lockout controls were configured:

| Policy                              |             Value |
| ----------------------------------- | ----------------: |
| Account lockout threshold           | 5 failed attempts |
| Account lockout duration            |        15 minutes |
| Reset account lockout counter after |        15 minutes |

### Security Rationale

The policy helps reduce the effectiveness of repeated password guessing attempts while keeping the lockout period short enough for a laboratory environment.

## Advanced Audit Policy

The following audit settings were enabled:

### Logon/Logoff

| Audit Policy                    |  Success |  Failure |
| ------------------------------- | -------: | -------: |
| Audit Logon                     |  Enabled |  Enabled |
| Audit Logoff                    |  Enabled | Disabled |
| Audit Account Lockout           | Disabled |  Enabled |
| Audit Special Logon             |  Enabled | Disabled |
| Audit Other Logon/Logoff Events |  Enabled |  Enabled |

### Account Management

| Audit Policy                        | Success | Failure |
| ----------------------------------- | ------: | ------: |
| Audit User Account Management       | Enabled | Enabled |
| Audit Security Group Management     | Enabled | Enabled |
| Audit Computer Account Management   | Enabled | Enabled |
| Audit Distribution Group Management | Enabled | Enabled |

### Policy Change

| Audit Policy        | Success | Failure |
| ------------------- | ------: | ------: |
| Audit Policy Change | Enabled | Enabled |

### System

| Audit Policy           | Success | Failure |
| ---------------------- | ------: | ------: |
| Audit System Integrity | Enabled | Enabled |

### Detailed Tracking

| Audit Policy              | Success |  Failure |
| ------------------------- | ------: | -------: |
| Audit Process Creation    | Enabled | Disabled |
| Audit Process Termination | Enabled | Disabled |

### Account Logon

| Audit Policy                             | Success | Failure |
| ---------------------------------------- | ------: | ------: |
| Audit Credential Validation              | Enabled | Enabled |
| Audit Kerberos Authentication Service    | Enabled | Enabled |
| Audit Kerberos Service Ticket Operations | Enabled | Enabled |

## Group Policy Application

The baseline is delivered through:

```text
Default Domain Policy
        ↓
corp.local
        ↓
SOC-WIN01
```

The computer-side policy application was validated with:

```cmd
gpresult /r /scope computer
```

The result confirmed:

```text
Applied Group Policy Objects
    Default Domain Policy
```

## Configuration Validation

The account and password policy values were validated on `SOC-WIN01` using:

```cmd
net accounts
```

The resulting values matched the configured baseline, including:

```text
Minimum password age        1 day
Maximum password age        60 days
Minimum password length     12
Password history            24
Lockout threshold           5
Lockout duration            15 minutes
Lockout observation window  15 minutes
```

## Result

The Windows endpoint now has a defined domain security baseline with:

* Stronger password requirements
* Account lockout protection
* Authentication auditing
* Account and group change auditing
* Process creation auditing
* Kerberos-related auditing
* Policy change auditing
* System integrity auditing

This provides the security foundation required for future SOC monitoring and detection projects without introducing those components into this project.

