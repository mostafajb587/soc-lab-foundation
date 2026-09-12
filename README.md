# Enterprise SOC Lab Foundation

A lightweight, enterprise-style cybersecurity lab designed from a SOC Analyst perspective and built as the foundation for future security monitoring, detection, investigation, threat hunting, incident response, and SOC automation projects.

The project focuses on building a realistic and controlled enterprise security foundation rather than implementing the full SOC stack in a single environment.

## Overview

The lab provides a segmented environment containing:

* pfSense firewall and gateway
* Separate Users and Servers networks
* Active Directory
* Internal DNS
* Domain-joined Windows endpoint
* Centralized Group Policy
* Basic Windows security baseline
* Controlled inter-network communication
* Validated firewall segmentation

The environment is intentionally lightweight so it can operate as a practical home lab while still demonstrating important enterprise security concepts.

## Project Scope

This project focuses on:

```text
Network Foundation
        ↓
Firewall & Segmentation
        ↓
DNS & Active Directory
        ↓
Windows Endpoint
        ↓
Security Baseline
        ↓
Validation
```

Advanced security monitoring and SOC tooling are intentionally excluded from this project.

Future projects may introduce:

* SIEM
* Endpoint telemetry
* Network monitoring
* Detection engineering
* Threat hunting
* Incident investigation
* Incident response
* SOAR and automation

## Architecture

The final lab architecture consists of three primary network zones:

* **WAN** — VMware NAT / Internet connectivity
* **USERS** — Windows user endpoint network
* **SERVERS** — Infrastructure and Active Directory network

```text
                         Internet
                            │
                            ▼
                     VMware NAT (VMnet8)
                            │
                            │ WAN
                            ▼
                    ┌───────────────┐
                    │    SOC-FW01   │
                    │    pfSense    │
                    │ Firewall / GW │
                    └───────┬───────┘
                            │
                  ┌─────────┴─────────┐
                  │                   │
               VMnet1              VMnet2
                USERS              SERVERS
           10.10.10.0/24       10.10.20.0/24
                  │                   │
                  ▼                   ▼
             SOC-WIN01            SOC-DC01
          10.10.10.128          10.10.20.10
          Windows Client        AD + DNS
```

### Architecture Diagram

![Enterprise SOC Lab Foundation](./diagram/network-topology.png)

> **Figure 1 — Final SOC Lab Foundation network architecture.**

## Core Components

| Component     | Role                                                      |
| ------------- | --------------------------------------------------------- |
| **SOC-FW01**  | pfSense firewall, gateway, routing, and security boundary |
| **SOC-WIN01** | Windows domain-joined user endpoint                       |
| **SOC-DC01**  | Windows Server providing Active Directory and DNS         |
| **VMnet8**    | VMware NAT / WAN connectivity                             |
| **VMnet1**    | Users network                                             |
| **VMnet2**    | Servers network                                           |

## Network Addressing

| Zone    | VMware Network | Network           | Gateway      | Primary System |
| ------- | -------------- | ----------------- | ------------ | -------------- |
| WAN     | VMnet8         | `192.168.46.0/24` | DHCP         | SOC-FW01       |
| USERS   | VMnet1         | `10.10.10.0/24`   | `10.10.10.1` | SOC-WIN01      |
| SERVERS | VMnet2         | `10.10.20.0/24`   | `10.10.20.1` | SOC-DC01       |

VMnet1 and VMnet2 are separate VMware virtual networks. They are used as logical network segments and are **not VLANs**.

## Network Flow

```text
Internet
   │
   ▼
VMware NAT / VMnet8
   │
   ▼
SOC-FW01
   │
   ├──────────────► USERS
   │                10.10.10.0/24
   │                SOC-WIN01
   │
   └──────────────► SERVERS
                    10.10.20.0/24
                    SOC-DC01
```

All inter-network traffic is evaluated by pfSense firewall policy.

## Firewall and Segmentation

pfSense provides the security boundary between the Users and Servers networks.

The final LAN policy is:

```text
USERS → SOC-DC01
        ALLOW

USERS → Other SERVERS
        BLOCK
```

The active policy contains:

| Order | Source     | Destination     | Action            |
| ----: | ---------- | --------------- | ----------------- |
|     1 | LAN subnet | `10.10.20.10`   | Allow             |
|     2 | LAN subnet | `10.10.20.0/24` | Block             |
|     3 | LAN subnet | Any             | Allow as required |

The IPv6 default allow rule was disabled because IPv6 is not part of the current lab design.

A temporary ICMP validation rule used during testing was disabled after validation and is not part of the final active policy.

### Firewall Interfaces

![pfSense interface assignments](./diagram/pfSense-interface-assignments.png)

> **Figure 2 — pfSense interface assignments for WAN, Users, and Servers networks.**

### Final Firewall Policy

![Final LAN firewall rules](./diagram/firewall-lan-final-rules.png)

> **Figure 3 — Final LAN firewall policy enforcing controlled Users-to-Servers communication.**

Detailed firewall documentation is available in:

[`docs/firewall.md`](./docs/firewall.md)

Detailed segmentation documentation is available in:

[`docs/segmentation.md`](./docs/segmentation.md)

## Active Directory

The lab uses:

```text
Domain: corp.local
NetBIOS: CORP
```

`SOC-DC01` provides:

* Active Directory Domain Services
* Internal DNS
* Domain authentication
* Kerberos-based authentication
* Centralized identity management

The Active Directory environment is organized into dedicated OUs and security groups.

### Active Directory Structure

![Active Directory structure](./diagram/active-directory-structure.png)

> **Figure 4 — Active Directory organizational structure.**

The Windows endpoint is joined to the `corp.local` domain and uses `SOC-DC01` for domain-related DNS and authentication services.

Detailed documentation:

[`docs/active-directory.md`](./docs/active-directory.md)

## Security Baseline

A domain-level security baseline was implemented through the **Default Domain Policy**.

### Password Policy

The effective baseline includes:

| Policy                  |   Value |
| ----------------------- | ------: |
| Minimum password length |      12 |
| Password complexity     | Enabled |
| Password history        |      24 |
| Maximum password age    | 60 days |
| Minimum password age    |   1 day |

### Account Lockout

| Policy              |             Value |
| ------------------- | ----------------: |
| Lockout threshold   | 5 failed attempts |
| Lockout duration    |        15 minutes |
| Reset counter after |        15 minutes |

### Security Auditing

The baseline also enables security-relevant auditing for:

* Logon and logoff activity
* Account lockouts
* Special logons
* User account management
* Security group management
* Computer account management
* Policy changes
* System integrity
* Process creation
* Kerberos authentication
* Kerberos service ticket activity

### Password Policy Configuration

![Password policy](./diagram/gpo-password-policy.png)

> **Figure 5 — Domain password security baseline configured through the Default Domain Policy.**

### Account Lockout Policy

![Account lockout policy](./diagram/gpo-account-lockout-policy.png)

> **Figure 6 — Domain account lockout policy configured through the Default Domain Policy.**

Detailed documentation:

[`docs/security-baseline.md`](./docs/security-baseline.md)

## Group Policy Validation

The Default Domain Policy was successfully applied to `SOC-WIN01`.

![Group Policy application](./diagram/gpresult-default-domain-policy.png)

> **Figure 7 — Successful application of the Default Domain Policy to SOC-WIN01.**

The effective password and lockout values were verified locally on the endpoint.

![Effective security baseline](./diagram/net-accounts-security-baseline.png)

> **Figure 8 — Effective password and account lockout settings on SOC-WIN01.**

## DNS

The Domain Controller provides internal DNS for the lab.

```text
SOC-DC01
10.10.20.10
```

The endpoint resolves:

```text
corp.local
→ 10.10.20.10
```

![DNS and domain validation](./diagram/dns-domain-validation.png)

> **Figure 10 — DNS resolution and Domain Controller discovery validation from SOC-WIN01.**

Detailed documentation:

[`docs/dns.md`](./docs/dns.md)

## Validation

The environment was validated after implementation.

### Network Validation

* Users network connectivity — **PASS**
* Servers network connectivity — **PASS**
* pfSense routing — **PASS**

### DNS Validation

```text
nslookup corp.local
```

Result:

```text
corp.local → 10.10.20.10
```

**PASS**

### Domain Controller Discovery

```text
nltest /dsgetdc:corp.local
```

**PASS**

### Secure Channel

```text
nltest /sc_verify:corp.local
```

Result:

```text
NERR_Success
```

**PASS**

### Domain Authentication

The endpoint successfully authenticated using the domain identity:

```text
CORP\Mostafa
```

**PASS**

### Group Policy

```text
gpresult /r /scope computer
```

Confirmed:

```text
Default Domain Policy
```

**PASS**

### Firewall Segmentation

Required Domain Controller access:

```text
SOC-WIN01
    ↓
10.10.20.10:389
    ↓
TcpTestSucceeded: True
```

**PASS**

Unauthorized access to the Servers gateway:

```text
SOC-WIN01
    ↓
10.10.20.1
    ↓
Request timed out
```

**PASS**

![Final segmentation validation](./diagram/final-segmentation-validation.png)

> **Figure 9 — Final firewall segmentation validation showing required Domain Controller access allowed while unauthorized access to the Servers network is blocked.**

Complete validation results:

[`docs/validation.md`](./docs/validation.md)

## Implementation

The project was implemented in the following sequence:

```text
1. VMware network preparation
        ↓
2. pfSense deployment
        ↓
3. Users / Servers network configuration
        ↓
4. Domain Controller deployment
        ↓
5. Active Directory and DNS
        ↓
6. Windows endpoint deployment
        ↓
7. Domain Join
        ↓
8. Security baseline
        ↓
9. Firewall segmentation
        ↓
10. Final validation
```

Implementation details:

[`docs/implementation.md`](./docs/implementation.md)

## Security Objectives

The completed environment provides:

* Network segmentation
* Controlled inter-network communication
* Centralized identity
* Domain authentication
* DNS-based domain discovery
* Password security controls
* Account lockout protection
* Windows security auditing
* Controlled Users-to-Servers access
* Validated firewall enforcement

## SOC Relevance

This lab provides multiple security telemetry foundations for future SOC projects.

### Network Security

Future projects can use the environment to investigate:

* Network scanning
* Unauthorized access attempts
* Cross-segment communication
* Lateral movement
* Firewall events
* Suspicious outbound connections

### Identity and Authentication

Active Directory provides a foundation for investigating:

* Successful logons
* Failed logons
* Account lockouts
* User and group changes
* Privileged activity
* Kerberos authentication activity

### Endpoint Telemetry

`SOC-WIN01` provides a realistic Windows endpoint for future monitoring and investigation activities.

### Future Projects

The foundation is intentionally separated from future security tooling.

Future projects may add:

```text
Project 01
SOC Lab Foundation
        ↓
Windows / Network Telemetry
        ↓
SIEM
        ↓
Detection Engineering
        ↓
Threat Hunting
        ↓
Incident Response
        ↓
SOAR / Automation
```

## Documentation

| Area              | Documentation                                              |
| ----------------- | ---------------------------------------------------------- |
| Network           | [`docs/network.md`](./docs/network.md)                     |
| Firewall          | [`docs/firewall.md`](./docs/firewall.md)                   |
| Segmentation      | [`docs/segmentation.md`](./docs/segmentation.md)           |
| DNS               | [`docs/dns.md`](./docs/dns.md)                             |
| Active Directory  | [`docs/active-directory.md`](./docs/active-directory.md)   |
| Security Baseline | [`docs/security-baseline.md`](./docs/security-baseline.md) |
| Implementation    | [`docs/implementation.md`](./docs/implementation.md)       |
| Validation        | [`docs/validation.md`](./docs/validation.md)               |

## Project Status

```text
STATUS: COMPLETE
```

The SOC Lab Foundation has been implemented, secured, tested, and documented.

The environment is ready to serve as the infrastructure foundation for the next SOC-focused project.
