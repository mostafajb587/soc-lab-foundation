# Implementation

## Objective

Document the major implementation stages used to build the SOC Lab Foundation.

The implementation follows the approved architecture and keeps the project focused on foundational enterprise infrastructure.

## 1. VMware Network Preparation

Three VMware networks were used:

```text
VMnet8 → WAN / Internet connectivity
VMnet1 → USERS network
VMnet2 → SERVERS network
```

VMnet1 and VMnet2 provide separate internal network segments for endpoints and infrastructure systems.

## 2. pfSense Deployment

`SOC-FW01` was deployed as the central firewall and router.

The firewall provides separate interfaces for:

```text
WAN
USERS
SERVERS
```

The internal gateways were configured as:

```text
USERS   → 10.10.10.1/24
SERVERS → 10.10.20.1/24
```

WAN connectivity is provided through VMware NAT.

## 3. Domain Controller Deployment

`SOC-DC01` was deployed in the Servers network with:

```text
IP Address: 10.10.20.10
```

Active Directory Domain Services and DNS were installed.

The internal domain was established as:

```text
corp.local
```

## 4. Active Directory Configuration

The Active Directory environment was organized into dedicated OUs:

```text
Corporate Users
Corporate Computers
Corporate Servers
Security Groups
IT Administration
```

Users and security groups were created according to their operational roles.

The following security groups were established:

```text
SOC-Analysts
IT-Admins
```

A standard user identity and a separate administrative identity were created.

## 5. Windows Endpoint Deployment

`SOC-WIN01` was deployed in the Users network with:

```text
IP Address: 10.10.10.128
Gateway:    10.10.10.1
DNS:        10.10.20.10
```

The endpoint was joined to:

```text
corp.local
```

The computer object was renamed to:

```text
SOC-WIN01
```

and moved into:

```text
Corporate Computers
```

## 6. Domain Authentication

The endpoint was successfully authenticated against the domain using:

```text
CORP\Mostafa
```

Domain functionality was validated using:

```cmd
whoami
nltest /dsgetdc:corp.local
nltest /sc_verify:corp.local
```

The tests confirmed successful domain authentication, Domain Controller discovery, and secure channel operation.

## 7. Security Baseline

A domain-level security baseline was configured through the `Default Domain Policy`.

The baseline includes:

* Password security
* Account lockout controls
* Authentication auditing
* Account management auditing
* Group management auditing
* Computer account auditing
* Policy change auditing
* System integrity auditing
* Process creation auditing
* Kerberos auditing

The detailed configuration is documented in:

```text
docs/security-baseline.md
```

## 8. Firewall Segmentation

The final firewall policy was configured on `SOC-FW01`.

The Users network is explicitly permitted to reach the Domain Controller while access to other destinations in the Servers subnet is blocked.

The final policy was validated through both allowed and blocked traffic tests.

Detailed firewall configuration is documented in:

```text
docs/firewall.md
docs/segmentation.md
```

## 9. Validation

The implementation was validated throughout the build using:

```text
DNS resolution
Domain Controller discovery
Secure channel verification
Domain authentication
Group Policy application
Password policy verification
Firewall allow testing
Firewall block testing
Network connectivity testing
```

The complete validation results are documented in:

```text
docs/validation.md
```

## 10. Implementation Result

The final environment consists of:

```text
Internet
   │
VMware NAT / VMnet8
   │
SOC-FW01
   ├── USERS
   │    └── SOC-WIN01
   │
   └── SERVERS
        └── SOC-DC01
```

The environment provides a functional enterprise-style foundation with:

* Network segmentation
* Firewall enforcement
* Centralized DNS
* Active Directory
* Domain authentication
* Organizational structure
* Security baseline
* Validated inter-network access control

The platform is intentionally prepared for future SOC projects without implementing future monitoring and detection components inside this project.
