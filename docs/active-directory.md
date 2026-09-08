
## 1. Active Directory Overview

The lab uses **SOC-DC01** as the central identity and infrastructure server.

It will provide:

* Active Directory Domain Services (AD DS).
* Internal DNS.
* User and computer authentication.
* Group-based authorization.
* Basic organizational structure.
* Security telemetry for future SOC investigations.

## 2. Active Directory Architecture

```text id="4qzq3u"
                 SERVERS
             10.10.20.0/24
                    │
                    ▼
              ┌───────────┐
              │ SOC-DC01  │
              │ Windows   │
              │ Server    │
              └─────┬─────┘
                    │
              AD DS + DNS
                    │
                    ▼
              ┌───────────┐
              │ SOC-WIN01 │
              │ Windows   │
              │ Client    │
              └───────────┘
```

## 3. Domain Controller

| Component  | Design                  |
| ---------- | ----------------------- |
| Hostname   | SOC-DC01                |
| Network    | SERVERS                 |
| IP Address | `10.10.20.10/24`        |
| Gateway    | `10.10.20.1`            |
| DNS        | `10.10.20.10`           |
| Role       | Domain Controller + DNS |

## 4. Domain Structure

The lab will use a single Active Directory domain with a simple OU structure.

Example:

```text id="t3l5oj"
Domain
│
├── Users
│
├── Computers
│
├── Servers
│
└── Groups
```

The structure is intentionally simple and focused on SOC-related identity monitoring rather than complex enterprise administration.

## 5. Identity and Authorization

The environment will use:

* Individual user accounts.
* Security groups.
* Group-based permissions.
* Separate administrative privileges.
* Least-privilege principles.

Users should receive only the permissions required for their intended role.

## 6. Windows Client Integration

**SOC-WIN01** will be joined to the Active Directory domain.

The client will use:

```text
IP Address : 10.10.10.10
Gateway    : 10.10.10.1
DNS        : 10.10.20.10
```

This allows centralized authentication and domain-based management.

## 7. Security Telemetry

Active Directory and Windows authentication will generate security-relevant events that can later be collected by the SOC.

Examples include:

| Activity                 | Example Event       |
| ------------------------ | ------------------- |
| Successful logon         | Event ID 4624       |
| Failed logon             | Event ID 4625       |
| Privileged logon         | Event ID 4672       |
| Process creation         | Event ID 4688       |
| User creation            | Event ID 4720       |
| User deletion            | Event ID 4726       |
| Group membership changes | Event IDs 4728/4732 |

These events will support future detection engineering, threat hunting, and incident investigation.

## 8. Security Objectives

The Active Directory design aims to:

* Centralize authentication.
* Apply basic least-privilege principles.
* Separate users, computers, servers, and groups logically.
* Generate useful Windows security telemetry.
* Provide a realistic identity layer for future SOC investigations.

## 9. Validation

The Active Directory implementation will be validated by confirming:

| Test                           | Expected Result              |
| ------------------------------ | ---------------------------- |
| DNS resolution                 | Successful                   |
| Domain Controller reachability | Successful                   |
| Client domain join             | Successful                   |
| Domain user authentication     | Successful                   |
| Group membership               | Correct                      |
| Authorization                  | Matches assigned permissions |
| Windows Security Events        | Generated as expected        |

## 10. SOC Relevance

Active Directory provides the identity layer of the SOC lab.

It enables future investigation of:

* Brute-force attempts.
* Credential abuse.
* Privilege escalation.
* Suspicious logons.
* Account creation.
* Group membership changes.
* Lateral movement.
* Compromised user accounts.

This makes the AD environment a key source of telemetry for future **SIEM, Detection Engineering, Threat Hunting, and Incident Response** projects.
