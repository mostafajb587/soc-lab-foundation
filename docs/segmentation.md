
## 1. Segmentation Overview

The lab separates endpoints and infrastructure services into different network segments.

The main purpose is to reduce unnecessary communication between systems and provide clear security boundaries for monitoring and investigation.

## 2. Network Segments

| Segment     | Subnet            | Main Components | Purpose                 |
| ----------- | ----------------- | --------------- | ----------------------- |
| **USERS**   | `10.10.10.0/24`   | SOC-WIN01       | User endpoint network   |
| **SERVERS** | `10.10.20.0/24`   | SOC-DC01        | Infrastructure services |
| **WAN**     | `192.168.46.0/24` | SOC-FW01 WAN    | External connectivity   |

## 3. Segmentation Boundary

```text
                 WAN
                  │
             ┌────▼─────┐
             │ SOC-FW01 │
             │ pfSense  │
             └────┬─────┘
                  │
          ┌───────┴───────┐
          │               │
       USERS           SERVERS
   10.10.10.0/24    10.10.20.0/24
          │               │
     SOC-WIN01        SOC-DC01
```

All traffic between the internal segments passes through **SOC-FW01**.

## 4. Communication Model

| Source               | Destination       | Requirement                          |
| -------------------- | ----------------- | ------------------------------------ |
| SOC-WIN01            | SOC-DC01          | Allowed for required AD/DNS services |
| SOC-WIN01            | Internet          | Allowed as required                  |
| USERS                | SERVERS           | Restricted by firewall policy        |
| Unauthorized traffic | Internal networks | Denied                               |

The exact firewall rules are documented separately in `firewall.md`.

## 5. Security Objectives

The segmentation design aims to:

* Reduce unnecessary east-west traffic.
* Limit unauthorized access to server resources.
* Create a controlled boundary between users and infrastructure.
* Reduce the potential impact of a compromised endpoint.
* Support investigation of lateral movement attempts.

## 6. Design Decision

VMware virtual networks are used to provide separate network segments.

The design does **not** use VLANs because the lab does not require physical switching infrastructure. Separate VMnet networks provide sufficient segmentation for this lightweight SOC environment.

## 7. SOC Relevance

Network segmentation provides useful security context for SOC operations:

| Security Area      | SOC Value                                                    |
| ------------------ | ------------------------------------------------------------ |
| Segmentation       | Defines trusted/untrusted boundaries                         |
| Firewall           | Provides allowed/blocked traffic telemetry                   |
| Users → Servers    | Helps identify unauthorized access                           |
| Lateral Movement   | Makes suspicious cross-segment traffic easier to investigate |
| Network Monitoring | Provides clear network zones for future detection rules      |
