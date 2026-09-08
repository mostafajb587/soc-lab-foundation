
## 1. Architecture Overview

The **Enterprise SOC Lab Foundation** is a lightweight, segmented enterprise-style environment designed to support future SOC monitoring, detection, investigation, threat hunting, and incident response projects.

The architecture uses **pfSense as the central security boundary**, separating external connectivity, user endpoints, and server infrastructure.

---

## 2. Design Goals

The architecture is designed around five main goals:

* **Segmentation** — Separate Users and Servers networks.
* **Security** — Control inter-network traffic through pfSense.
* **Simplicity** — Avoid unnecessary enterprise complexity.
* **Resource Efficiency** — Keep the lab suitable for limited hardware resources.
* **SOC Readiness** — Provide realistic infrastructure for future security telemetry and investigations.

---

## 3. Logical Architecture

```text
                         Internet
                            │
                     VMware NAT / VMnet8
                            │
                            ▼
                    ┌───────────────┐
                    │    SOC-FW01   │
                    │    pfSense    │
                    │ Firewall / GW │
                    └───────┬───────┘
                            │
                  ┌─────────┴─────────┐
                  │                   │
             USERS NETWORK        SERVER NETWORK
              VMnet1                VMnet2
           10.10.10.0/24         10.10.20.0/24
                  │                   │
             SOC-WIN01            SOC-DC01
             Windows Client       Windows Server
                                  AD + DNS
```

All internal communication between the Users and Servers networks passes through pfSense.

---

## 4. Network Zones

| Zone    | Network           | VMware | Purpose                 |
| ------- | ----------------- | ------ | ----------------------- |
| WAN     | `192.168.46.0/24` | VMnet8 | External connectivity   |
| USERS   | `10.10.10.0/24`   | VMnet1 | User endpoints          |
| SERVERS | `10.10.20.0/24`   | VMnet2 | Infrastructure services |

The Users and Servers zones are intentionally isolated into separate virtual networks.

---

## 5. Component Roles

| Component     | Role                                              |
| ------------- | ------------------------------------------------- |
| **SOC-FW01**  | Firewall, gateway, routing, and security boundary |
| **SOC-WIN01** | User workstation / Windows endpoint               |
| **SOC-DC01**  | Active Directory Domain Controller + DNS          |
| **VMnet8**    | WAN / Internet connectivity                       |
| **VMnet1**    | Users network                                     |
| **VMnet2**    | Servers network                                   |

---

## 6. Traffic Flow

```text
Internet
   │
   ▼
VMnet8
   │
   ▼
pfSense
   │
   ├──► USERS
   │      │
   │      └──► SOC-WIN01
   │
   └──► SERVERS
          │
          └──► SOC-DC01
               AD + DNS
```

The firewall controls traffic between network zones and provides the routing point for internal communication.

---

## 7. Security Boundaries

The primary security boundary is the **pfSense firewall**.

```text
              TRUST BOUNDARY
                    │
                    ▼
        ┌───────────────────────┐
        │       pfSense         │
        └───────────┬───────────┘
                    │
          ┌─────────┴─────────┐
          │                   │
       USERS               SERVERS
     Less Trusted        More Trusted
```

The design allows security policies to restrict unnecessary access from the Users network to the Servers network while permitting required services such as DNS and domain authentication.

---

## 8. Design Decisions

| Decision                            | Reason                                        |
| ----------------------------------- | --------------------------------------------- |
| Separate Users and Servers networks | Establish network segmentation                |
| pfSense as central gateway          | Centralize routing and security policies      |
| VMware NAT for WAN                  | Provide controlled external connectivity      |
| Static internal addressing          | Simplify infrastructure management            |
| Single Domain Controller            | Sufficient for a lightweight home lab         |
| No VLAN infrastructure              | Avoid unnecessary switching complexity        |
| No HA / redundant infrastructure    | Not required for the project's SOC objectives |

The architecture prioritizes practical SOC learning over production-scale infrastructure complexity.

---

## 9. SOC Relevance

This architecture provides the foundation for future security monitoring and investigation activities.

| Component            | Future SOC Value                                                 |
| -------------------- | ---------------------------------------------------------------- |
| pfSense              | Firewall events, blocked/allowed traffic, network reconnaissance |
| Active Directory     | Authentication, privilege, account, and group activity           |
| Windows Endpoint     | Process, network, and endpoint telemetry                         |
| DNS                  | Name-resolution activity and future DNS security monitoring      |
| Network Segmentation | Investigation of lateral movement and unauthorized access        |

The foundation will be extended by future projects that introduce centralized logging, endpoint telemetry, detection engineering, threat hunting, and incident response capabilities.
