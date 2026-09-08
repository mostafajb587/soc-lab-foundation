
## 1. Firewall Overview

**SOC-FW01** runs pfSense and acts as the central security boundary between the WAN, Users, and Servers networks.

Its main responsibilities are:

* Routing between network segments.
* Controlling inter-network communication.
* Controlling outbound Internet access.
* Enforcing security policies.
* Providing firewall telemetry for future SOC monitoring.

## 2. Firewall Interfaces

| Interface   | Network | Address         | Role                  |
| ----------- | ------- | --------------- | --------------------- |
| **WAN**     | VMnet8  | DHCP            | External connectivity |
| **USERS**   | VMnet1  | `10.10.10.1/24` | Users gateway         |
| **SERVERS** | VMnet2  | `10.10.20.1/24` | Servers gateway       |

## 3. Traffic Control Model

```text
                 Internet
                    │
                    ▼
              ┌───────────┐
              │ SOC-FW01  │
              │  pfSense  │
              └─────┬─────┘
                    │
           ┌────────┴────────┐
           │                 │
        USERS             SERVERS
     10.10.10.0/24     10.10.20.0/24
           │                 │
      SOC-WIN01          SOC-DC01
```

All traffic crossing between these zones is evaluated by pfSense firewall policy.

## 4. Security Policy Model

The firewall follows a **least-privilege** approach:

| Traffic                          | Policy Intent     |
| -------------------------------- | ----------------- |
| USERS → Required AD/DNS services | Allow             |
| USERS → Internet                 | Allow as required |
| USERS → Other server services    | Restrict          |
| SERVERS → Internet               | Allow as required |
| Unauthorized inter-zone traffic  | Deny              |
| Unnecessary inbound WAN traffic  | Deny              |

The final implementation will use explicit firewall rules based on these requirements.

## 5. Security Objectives

The firewall is designed to:

* Prevent unauthorized network access.
* Restrict unnecessary communication between Users and Servers.
* Reduce lateral movement opportunities.
* Control external connectivity.
* Provide a central point for network security logging.
* Support future detection and investigation activities.

## 6. Logging and Monitoring

Firewall events will provide useful telemetry for future SOC operations, including:

* Allowed connections.
* Blocked connections.
* Source and destination IP addresses.
* Source and destination ports.
* Protocol information.
* Repeated connection attempts.

This telemetry can later be forwarded to a SIEM such as Splunk for detection and investigation.

## 7. Design Decisions

* pfSense is used as the central firewall and routing device.
* Internal traffic must pass through pfSense between network segments.
* The design follows least privilege rather than unrestricted internal communication.
* No advanced firewall features are required at this stage.
* Firewall configuration will be validated after implementation.

## 8. SOC Relevance

The firewall is a key security telemetry source in the lab.

It can help a SOC Analyst identify:

* Port scanning.
* Unauthorized access attempts.
* Suspicious outbound connections.
* Cross-segment communication.
* Potential lateral movement.
* Repeated blocked connection attempts.
