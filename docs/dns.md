# DNS

## Objective

Provide centralized name resolution for the lab and support Active Directory domain discovery and authentication.

DNS is hosted on the Domain Controller and is used by domain-joined Windows systems.

## DNS Server

The primary DNS server in the lab is:

```text
SOC-DC01
10.10.20.10
```

The server is located in the **SERVERS** network:

```text
10.10.20.0/24
```

`SOC-WIN01` uses `10.10.20.10` as its DNS server.

## Domain

The Active Directory domain is:

```text
corp.local
```

DNS resolution for the domain is handled by the Domain Controller.

```text
SOC-WIN01
10.10.10.128
      │
      │ DNS
      ▼
SOC-DC01
10.10.20.10
      │
      ▼
corp.local
```

## Role in Active Directory

DNS is a critical dependency for Active Directory.

The Windows endpoint uses DNS to discover domain services and locate the Domain Controller.

Without correct DNS configuration, domain join, authentication, and domain service discovery may fail even when basic IP connectivity is working.

## Network Placement

DNS is intentionally hosted in the Servers network together with the Domain Controller.

| Component      | Address        | Network         |
| -------------- | -------------- | --------------- |
| SOC-DC01 / DNS | `10.10.20.10`  | `10.10.20.0/24` |
| SOC-WIN01      | `10.10.10.128` | `10.10.10.0/24` |

The firewall permits the required communication between the Users network and the Domain Controller.

## Client Configuration

`SOC-WIN01` is configured to use:

```text
Preferred DNS Server:
10.10.20.10
```

This allows the endpoint to resolve:

```text
corp.local
```

and discover domain services correctly.

## DNS Validation

DNS resolution was tested from `SOC-WIN01` using:

```cmd
nslookup corp.local
```

The result returned:

```text
Name:
corp.local

Address:
10.10.20.10
```

This confirms that the domain name resolves to the expected Domain Controller.

## Network Connectivity Validation

DNS connectivity through the firewall was also validated using:

```powershell
Test-NetConnection 10.10.20.10 -Port 53
```

The result was:

```text
TcpTestSucceeded: True
```

This confirms that the Users network can reach the DNS service hosted on `SOC-DC01`.

## Security Considerations

DNS is restricted to the internal Active Directory architecture.

The Domain Controller remains inside the Servers network rather than being directly exposed to the external WAN.

The firewall controls communication between the Users and Servers networks.

## SOC Relevance

DNS will later become an important source of security telemetry for SOC operations.

Future projects may use DNS activity to investigate:

* Suspicious domain resolution
* Malware-related domains
* Command-and-control activity
* Unusual DNS behavior
* Internal reconnaissance

DNS monitoring and analysis are intentionally outside the scope of this project.

## Result

The lab has a functional centralized DNS service integrated with Active Directory.

Validated capabilities include:

* Domain name resolution
* DNS connectivity across the firewall
* Domain Controller discovery support
* Windows domain operation
