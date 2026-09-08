
## Network Overview

The lab uses three virtual networks:

* **VMnet8** — WAN / external connectivity
* **VMnet1** — Users network
* **VMnet2** — Servers network

pfSense connects these networks and provides the routing point between them.

## IP Addressing Plan

| Device / Interface   | Network | IP Address       | Purpose                   |
| -------------------- | ------- | ---------------- | ------------------------- |
| **SOC-FW01 WAN**     | VMnet8  | DHCP             | External/WAN connectivity |
| **SOC-FW01 USERS**   | VMnet1  | `10.10.10.1/24`  | Users gateway             |
| **SOC-FW01 SERVERS** | VMnet2  | `10.10.20.1/24`  | Servers gateway           |
| **SOC-WIN01**        | USERS   | `10.10.10.10/24` | Windows client            |
| **SOC-DC01**         | SERVERS | `10.10.20.10/24` | AD + DNS server           |

### Network Summary

| Network | Subnet            | Gateway      | Main Role               |
| ------- | ----------------- | ------------ | ----------------------- |
| WAN     | `192.168.46.0/24` | VMware NAT   | Internet access         |
| USERS   | `10.10.10.0/24`   | `10.10.10.1` | User endpoints          |
| SERVERS | `10.10.20.0/24`   | `10.10.20.1` | Infrastructure services |

## Network Topology

```text
                         Internet
                            │
                         VMnet8
                    192.168.46.0/24
                            │
                       pfSense WAN
                            │
                    ┌───────┴───────┐
                    │               │
                 VMnet1           VMnet2
               USERS             SERVERS
            10.10.10.0/24      10.10.20.0/24
                    │               │
               SOC-WIN01        SOC-DC01
              10.10.10.10      10.10.20.10
                                  AD + DNS
```

## Routing

pfSense acts as the default gateway for both internal networks:

```text
USERS   → 10.10.10.1
SERVERS → 10.10.20.1
```

Traffic between the Users and Servers networks is routed through pfSense, where firewall policies will determine whether the communication is permitted.

## DNS

The internal DNS server will be provided by **SOC-DC01**:

```text
SOC-WIN01
    │
    └── DNS → 10.10.20.10
                   │
                SOC-DC01
```

This allows the Windows client to resolve internal domain resources and supports Active Directory authentication.

## Design Notes

* Internal networks use dedicated private address ranges independent of the VMware NAT subnet.
* Static addressing is used for core infrastructure components.
* The WAN interface receives its address through VMware NAT.
* Users and Servers are maintained as separate network segments.
* Detailed firewall policies are documented separately in `firewall.md`.

## SOC Relevance

The network design provides clear boundaries for future security monitoring and investigation.

| Network Element | SOC Value                               |
| --------------- | --------------------------------------- |
| pfSense         | Firewall and network security telemetry |
| Users network   | Endpoint and user activity              |
| Servers network | Infrastructure and identity activity    |
| DNS             | Future DNS monitoring and investigation |
| Segmentation    | Lateral movement detection and analysis |
