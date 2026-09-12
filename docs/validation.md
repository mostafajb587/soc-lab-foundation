# Validation

## Objective

Validate that the SOC Lab Foundation is operational, correctly segmented, securely configured, and ready to support future security testing and SOC projects.

Validation was performed after the main infrastructure and security configuration were completed.

## 1. Network Validation

### Users Network

```text
Network: 10.10.10.0/24
Gateway: 10.10.10.1
```

`SOC-WIN01` was successfully placed in the Users network:

```text
10.10.10.128
```

### Servers Network

```text
Network: 10.10.20.0/24
Gateway: 10.10.20.1
```

`SOC-DC01` was successfully placed in the Servers network:

```text
10.10.20.10
```

## 2. DNS Validation

DNS resolution was tested from `SOC-WIN01`:

```cmd
nslookup corp.local
```

Result:

```text
corp.local
→ 10.10.20.10
```

Status:

```text
PASS
```

This confirms that the endpoint can resolve the Active Directory domain through the Domain Controller DNS service.

## 3. Domain Controller Discovery

Domain Controller discovery was tested with:

```cmd
nltest /dsgetdc:corp.local
```

Result:

```text
Command completed successfully
```

Status:

```text
PASS
```

## 4. Domain Authentication

The endpoint was authenticated using the domain account:

```text
CORP\mustafa
```

The identity was verified using:

```cmd
whoami
```

Result:

```text
corp\mustafa
```

Status:

```text
PASS
```

## 5. Secure Channel Validation

The secure channel between `SOC-WIN01` and the Active Directory domain was tested using:

```cmd
nltest /sc_verify:corp.local
```

Result:

```text
NERR_Success
```

for the Domain Controller connection and trust verification.

Status:

```text
PASS
```

## 6. Group Policy Validation

The computer-side Group Policy configuration was validated using:

```cmd
gpresult /r /scope computer
```

The result confirmed:

```text
Applied Group Policy Objects

Default Domain Policy
```

Status:

```text
PASS
```

## 7. Security Baseline Validation

The effective account and password policies were checked using:

```cmd
net accounts
```

The final values matched the configured baseline:

| Policy                     | Effective Value | Status |
| -------------------------- | --------------: | ------ |
| Minimum password age       |           1 day | PASS   |
| Maximum password age       |         60 days | PASS   |
| Minimum password length    |              12 | PASS   |
| Password history           |              24 | PASS   |
| Lockout threshold          |      5 attempts | PASS   |
| Lockout duration           |      15 minutes | PASS   |
| Lockout observation window |      15 minutes | PASS   |

## 8. Firewall Validation

### Allowed Communication

Required communication from the Users network to the Domain Controller was tested:

```powershell
Test-NetConnection 10.10.20.10 -Port 389
```

Result:

```text
TcpTestSucceeded: True
```

Status:

```text
PASS
```

This confirms that required communication to the Domain Controller remains available.

### Blocked Communication

Access from the Users network to the Servers gateway was tested:

```text
ping 10.10.20.1
```

Result:

```text
Request timed out
```

Status:

```text
PASS
```

This confirms that the final firewall segmentation prevents the tested Users-to-Servers traffic.

## 9. Segmentation Validation

The final traffic model was verified:

```text
USERS → SOC-DC01
        ALLOWED ✅

USERS → Other SERVERS
        BLOCKED ✅
```

The firewall therefore enforces the intended security boundary between the Users and Servers networks.

## 10. Temporary Rule Cleanup

A temporary ICMP validation rule was created on the Servers interface during early testing:

```text
Allow ICMP from host for validation
```

The rule was disabled after validation.

It is not part of the final active security policy.

## 11. Final Validation Summary

| Category                    | Status |
| --------------------------- | ------ |
| VMware network separation   | PASS   |
| pfSense routing             | PASS   |
| DNS resolution              | PASS   |
| Domain Controller discovery | PASS   |
| Domain authentication       | PASS   |
| Secure channel              | PASS   |
| Group Policy application    | PASS   |
| Password security baseline  | PASS   |
| Account lockout policy      | PASS   |
| Firewall allow policy       | PASS   |
| Firewall block policy       | PASS   |
| Network segmentation        | PASS   |
| Temporary test-rule cleanup | PASS   |

## 12. Final Result

The SOC Lab Foundation has been successfully implemented and validated.

The final environment provides:

* Segmented Users and Servers networks
* Centralized firewall enforcement
* Functional Active Directory
* Integrated DNS
* Domain-joined Windows endpoint
* Centralized Group Policy
* Account security baseline
* Validated inter-network access control

The environment is ready for future security testing, monitoring, detection, and investigation projects.

Those future components are intentionally excluded from this project to preserve the defined project scope.
