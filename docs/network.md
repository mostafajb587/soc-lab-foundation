# Network Design

## Objective

Define the final network architecture, IP addressing, VMware network mapping, and device placement for the SOC Lab Foundation.

The design separates user endpoints from server infrastructure and uses pfSense as the routing and security boundary between the networks.

## Network Architecture

```text
                         Internet
                            │
                            ▼
                     VMware NAT (VMnet8)
                            │
                            ▼
                       SOC-FW01
                         pfSense
                       /        \
                      /          \
                     ▼            ▼
                VMnet1          VMnet2
                 USERS          SERVERS
            10.10.10.0/24    10.10.20.0/24
                  │                │
                  ▼                ▼
             SOC-WIN01         SOC-DC01
            10.10.10.128      10.10.20.10
```

## VMware Networks

| VMware Network | Type / Role | Purpose                          |
| -------------- | ----------- | -------------------------------- |
| VMnet8         | NAT         | External / Internet connectivity |
| VMnet1         | Host-only   | Internal Users network           |
| VMnet2         | Host-only   | Internal Servers network         |

VMnet1 and VMnet2 are separate VMware virtual networks. They are used as internal segmentation boundaries and are **not VLANs**.

The current lab uses network separation through distinct VMware networks and Layer 3 firewall enforcement on pfSense.

## IP Addressing

### Users Network

```text
Network:  10.10.10.0/24
Gateway:  10.10.10.1
```

Primary endpoint:

```text
SOC-WIN01
IP: 10.10.10.128
```

### Servers Network

```text
Network:  10.10.20.0/24
Gateway:  10.10.20.1
```

Primary server:

```text
SOC-DC01
IP: 10.10.20.10
```

## Network Components

| Component | Network          | IP Address     | Role                    |
| --------- | ---------------- | -------------- | ----------------------- |
| SOC-FW01  | WAN / VMnet8     | DHCP           | WAN connectivity        |
| SOC-FW01  | USERS / VMnet1   | `10.10.10.1`   | Users gateway           |
| SOC-FW01  | SERVERS / VMnet2 | `10.10.20.1`   | Servers gateway         |
| SOC-WIN01 | USERS            | `10.10.10.128` | Windows endpoint        |
| SOC-DC01  | SERVERS          | `10.10.20.10`  | Domain Controller / DNS |

## Routing

Inter-network routing is provided by `SOC-FW01`.

Traffic between:

```text
10.10.10.0/24
        ↕
10.10.20.0/24
```

passes through pfSense and is evaluated by the firewall policy.

Direct Layer 2 communication between the Users and Servers networks is not present.

## DNS

The Active Directory DNS server is:

```text
10.10.20.10
```

`SOC-WIN01` uses this address for domain-related DNS resolution.

See:

```text
docs/dns.md
```

for detailed DNS documentation.

## Network Segmentation

The Users and Servers networks are intentionally separated.

```text
USERS
10.10.10.0/24
      │
      │ pfSense
      │
      ▼
SERVERS
10.10.20.0/24
```

The firewall policy permits the communication required by the domain environment while blocking access from Users to other server destinations.

This reduces unnecessary lateral connectivity between endpoints and infrastructure systems.

## Connectivity Validation

The network was validated using multiple tests.

### Domain Controller Discovery

```cmd
nltest /dsgetdc:corp.local
```

Result:

```text
Command completed successfully
```

### DNS Resolution

```cmd
nslookup corp.local
```

Result:

```text
corp.local
→ 10.10.20.10
```

### Secure Channel

```cmd
nltest /sc_verify:corp.local
```

Result:

```text
NERR_Success
```

### Inter-network Service Connectivity

From `SOC-WIN01`:

```powershell
Test-NetConnection 10.10.20.10 -Port 389
```

Result:

```text
TcpTestSucceeded: True
```

This confirms that required communication from the Users network to the Domain Controller is available.

### Segmentation Validation

From `SOC-WIN01`:

```text
ping 10.10.20.1
```

Result:

```text
Request timed out
```

This confirms that access to the Servers gateway is blocked by the final firewall policy.

## Security Considerations

The network design follows these principles:

* Users and Servers are separated into different network segments.
* The Domain Controller is isolated inside the Servers network.
* Inter-segment traffic is routed through pfSense.
* Firewall rules control communication between the segments.
* Administrative and infrastructure services are not placed directly on the Users network.
* No unnecessary VLAN complexity is introduced into the current VMware-based lab.

## Result

The final network design provides:

* Internet connectivity through VMware NAT
* Separate Users and Servers networks
* Centralized routing through pfSense
* Domain services isolated in the Servers network
* Validated inter-network communication
* Validated segmentation enforcement

This network foundation is ready for future monitoring, detection, and security-analysis projects.
