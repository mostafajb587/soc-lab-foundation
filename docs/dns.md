
## 1. DNS Overview

The lab uses **SOC-DC01** as the internal DNS server.

DNS is required for reliable internal name resolution and is an essential dependency for the Active Directory environment.

## 2. DNS Architecture

```text id="j4k8zn"
                    USERS
                 10.10.10.0/24
                       │
                  SOC-WIN01
                       │
                       │ DNS
                       ▼
                10.10.20.10
                       │
                       ▼
                  SOC-DC01
                  AD + DNS
                       │
                       ▼
                 DNS Forwarders
                       │
                   Internet
```

## 3. DNS Configuration

| Component      | Value                           |
| -------------- | ------------------------------- |
| DNS Server     | SOC-DC01                        |
| DNS IP         | `10.10.20.10`                   |
| DNS Type       | Internal DNS                    |
| Primary Client | SOC-WIN01                       |
| Client DNS     | `10.10.20.10`                   |
| Main Purpose   | AD and internal name resolution |

## 4. Internal Name Resolution

The internal DNS server will provide name resolution for the Active Directory environment.

Example:

```text
SOC-DC01 → 10.10.20.10
```

The Windows client will use the internal DNS server rather than relying directly on public DNS for domain-related resolution.

## 5. DNS and Active Directory

Active Directory depends heavily on DNS.

DNS will support:

* Domain discovery.
* Domain controller discovery.
* Kerberos-related services.
* LDAP-related services.
* Internal host resolution.
* Domain joining.

Therefore, **SOC-DC01 provides both AD DS and DNS services** in this lab.

## 6. External Resolution

For domains that are not hosted internally, the internal DNS server may use configured DNS forwarders to resolve external names.

This keeps client DNS configuration centralized and allows DNS activity to be monitored from the internal DNS server.

## 7. Security Considerations

The DNS design follows these principles:

* Clients use the internal DNS server for domain resolution.
* Direct dependency on public DNS from domain clients is avoided where possible.
* DNS is centralized on the domain controller.
* DNS configuration will be validated after implementation.
* DNS telemetry can be used for future security monitoring.

## 8. Validation

DNS functionality will be validated by confirming:

| Test                         | Expected Result                          |
| ---------------------------- | ---------------------------------------- |
| Client → DNS server          | Reachable                                |
| Internal hostname resolution | Successful                               |
| AD domain resolution         | Successful                               |
| External hostname resolution | Successful when forwarding is configured |
| Client domain join           | Successful                               |

## 9. SOC Relevance

DNS can provide valuable security telemetry for future SOC operations.

Potential investigation areas include:

* Suspicious domain lookups.
* Repeated failed DNS queries.
* Unusual external domains.
* Potential command-and-control activity.
* Malware-related DNS behavior.
* DNS-based tunneling.

DNS logs can later be integrated into the SIEM for detection and threat hunting.
