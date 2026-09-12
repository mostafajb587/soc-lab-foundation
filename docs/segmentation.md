# Network Segmentation

## Objective

Separate user endpoints from server infrastructure and enforce controlled communication between the two security zones.

The segmentation design is intended to reduce unnecessary lateral movement and establish a clear security boundary between endpoint systems and infrastructure services.

## Segmentation Model

The lab contains two isolated internal network segments:

```text id="vp5f4a"
USERS
10.10.10.0/24
    │
    │
    │ pfSense
    │
    ▼
SERVERS
10.10.20.0/24
```

### Users Network

```text id="4r47sm"
Network: 10.10.10.0/24
Gateway: 10.10.10.1
```

Primary endpoint:

```text id="z4rj7m"
SOC-WIN01
10.10.10.128
```

### Servers Network

```text id="gzqtj5"
Network: 10.10.20.0/24
Gateway: 10.10.20.1
```

Primary infrastructure server:

```text id="eg8o0y"
SOC-DC01
10.10.20.10
```

## VMware Segmentation

The two internal zones use separate VMware virtual networks:

| Network | VMware Network | Purpose                         |
| ------- | -------------- | ------------------------------- |
| USERS   | VMnet1         | Endpoint network                |
| SERVERS | VMnet2         | Server / infrastructure network |

VMnet1 and VMnet2 are separate VMware networks and are not physical VLANs.

The segmentation is implemented through separate Layer 3 interfaces on pfSense.

## Security Boundary

`SOC-FW01` acts as the security boundary between the two zones.

Traffic between the Users and Servers networks must traverse pfSense:

```text id="z3ik3o"
SOC-WIN01
10.10.10.128
      │
      ▼
SOC-FW01
pfSense
      │
      ▼
SOC-DC01
10.10.20.10
```

This allows the firewall to apply security policy before traffic reaches the destination network.

## Access Model

The final security model follows controlled access rather than unrestricted inter-network communication.

### Allowed

```text id="j3wq9m"
USERS → SOC-DC01
```

The Domain Controller is explicitly reachable because the Windows endpoint depends on infrastructure services such as DNS and Active Directory authentication.

### Blocked

```text id="d7j5pf"
USERS → Other SERVERS
```

Traffic from the Users network to other destinations in the Servers subnet is blocked.

This prevents a user endpoint from receiving unrestricted access to server infrastructure.

## Final Firewall Enforcement

The active LAN policy includes:

| Priority | Source     | Destination     | Action            |
| -------: | ---------- | --------------- | ----------------- |
|        1 | LAN subnet | `10.10.20.10`   | Allow             |
|        2 | LAN subnet | `10.10.20.0/24` | Block             |
|        3 | LAN subnet | Any             | Allow as required |

The explicit Domain Controller allow rule is evaluated before the broader Servers subnet block.

This ensures that required Active Directory communication remains available while other server destinations remain restricted.

## Validation

### Allowed Traffic

From `SOC-WIN01`, connectivity to the Domain Controller was tested:

```powershell id="1to1c6"
Test-NetConnection 10.10.20.10 -Port 389
```

Result:

```text id="4we0k5"
TcpTestSucceeded: True
```

The required communication to the Domain Controller is therefore allowed.

### Blocked Traffic

Connectivity from `SOC-WIN01` to the Servers gateway was tested:

```text id="up4ybl"
ping 10.10.20.1
```

Result:

```text id="5n8e16"
Request timed out
```

This confirms that the firewall blocks the tested Users-to-Servers traffic.

### Validation Rule Cleanup

A temporary ICMP rule was used during early firewall validation:

```text id="0g7hqa"
Allow ICMP from host for validation
```

The rule was disabled after testing and is not part of the final active policy.

## Security Rationale

Network segmentation provides several security benefits:

* Reduces unnecessary connectivity between endpoints and infrastructure.
* Limits lateral movement opportunities.
* Creates a clear security boundary.
* Centralizes inter-network access control at pfSense.
* Makes allowed communication explicit.
* Provides a foundation for future network monitoring and detection.

## SOC Relevance

Segmentation is important from a SOC perspective because unexpected cross-segment communication can become a useful detection signal.

Future security monitoring projects may use the segmentation boundary to identify:

* Unauthorized server access
* Port scanning
* Lateral movement
* Repeated blocked connections
* Suspicious endpoint-to-server communication

These monitoring and detection capabilities are outside the scope of the current project.

## Result

The lab now implements and validates a functional security boundary between Users and Servers.

The final design provides:

* Separate VMware networks
* Layer 3 separation through pfSense
* Explicit Domain Controller access
* Restricted Users-to-Servers communication
* Validated allow and deny behavior

The segmentation foundation is ready for future SOC monitoring and detection projects.
