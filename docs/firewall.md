# Firewall and Network Segmentation

## Objective

Implement network segmentation between the Users and Servers networks using pfSense and enforce controlled communication between them.

The firewall acts as the Layer 3 security boundary between:

* Users Network: `10.10.10.0/24`
* Servers Network: `10.10.20.0/24`

## Firewall Role

`SOC-FW01` provides:

* Inter-network routing
* Network isolation
* Traffic filtering
* Internet connectivity for the lab
* Security enforcement between Users and Servers

```text
Internet
   │
VMware NAT / VMnet8
   │
SOC-FW01
   ├── USERS
   │   10.10.10.0/24
   │
   └── SERVERS
       10.10.20.0/24
```

## Interface Design

| Interface | Network             | Gateway / Address | Purpose                |
| --------- | ------------------- | ----------------- | ---------------------- |
| WAN       | VMware NAT / VMnet8 | DHCP              | External connectivity  |
| LAN       | `10.10.10.0/24`     | `10.10.10.1`      | User endpoints         |
| SERVERS   | `10.10.20.0/24`     | `10.10.20.1`      | Infrastructure servers |

## Final LAN Policy

The final active LAN policy is ordered as follows:

| Order | Source                          | Destination       | Action | Purpose                                                    |
| ----: | ------------------------------- | ----------------- | ------ | ---------------------------------------------------------- |
|     1 | Firewall anti-lockout mechanism | LAN address       | Allow  | Preserve administrative access to pfSense                  |
|     2 | `LAN subnets`                   | `10.10.20.10`     | Allow  | Permit Users to reach the Domain Controller                |
|     3 | `LAN subnets`                   | `SERVERS subnets` | Block  | Prevent access to other Servers                            |
|     4 | `LAN subnets`                   | Any               | Allow  | Provide required access to permitted external destinations |

The IPv6 default allow rule was disabled because IPv6 is not part of the current lab design.

## Segmentation Model

The security boundary is implemented from the Users interface.

```text
USERS
10.10.10.0/24
      │
      ├──────────────→ 10.10.20.10
      │                SOC-DC01
      │                ALLOWED
      │
      └──────────────→ 10.10.20.0/24
                       OTHER SERVERS
                       BLOCKED
```

This provides isolation while preserving the communication required by the current Active Directory environment.

## Domain Controller Exception

`SOC-DC01` is explicitly allowed from the Users network because it provides core services required by the Windows domain environment, including DNS and Active Directory authentication services.

The current lab uses:

```text
SOC-DC01
10.10.20.10
```

## Temporary Validation Rule

A temporary ICMP rule was created on the Servers interface during testing:

```text
Allow ICMP from host for validation
```

The rule was disabled after validation and is not part of the final active firewall policy.

## Validation

### Allowed Traffic

The following connection was successfully validated from `SOC-WIN01`:

```text
Source:
10.10.10.128

Destination:
10.10.20.10:389
```

Result:

```text
TcpTestSucceeded: True
```

This confirms that the required communication path to the Domain Controller remains available.

### Blocked Traffic

Connectivity from `SOC-WIN01` to the Servers gateway was tested:

```text
Source:
10.10.10.128

Destination:
10.10.20.1
```

Result:

```text
Request timed out
```

This confirms that traffic blocked by the segmentation policy does not reach the Servers network gateway.

## Security Considerations

The firewall policy follows a controlled-access model rather than relying only on network separation.

The Users network is not given unrestricted access to the Servers network. Required access to the Domain Controller is explicitly permitted, while access to other server addresses is blocked.

The project intentionally does not implement advanced network security technologies such as IDS/IPS, network monitoring, or SIEM integration. These belong to later projects.

## Result

The pfSense firewall successfully enforces the intended Users-to-Servers security boundary.

The final environment provides:

* Separate Users and Servers networks
* Controlled inter-network routing
* Explicit Domain Controller access
* Blocking of unauthorized Users-to-Servers traffic
* Validated firewall enforcement
* Removal of temporary testing access
