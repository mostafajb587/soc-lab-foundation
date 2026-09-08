# Enterprise SOC Lab Foundation

## Overview

The **Enterprise SOC Lab Foundation** is a hands-on cybersecurity lab designed to build a small, segmented enterprise-style environment that serves as the foundation for future SOC monitoring, detection, investigation, threat hunting, and incident response projects.

The lab focuses on establishing the core infrastructure and security boundaries required for meaningful security telemetry and realistic attack simulations. It includes separate user and server networks, a pfSense firewall as the central security boundary, internal DNS, Active Directory, and Windows endpoints.

The environment is intentionally designed to remain lightweight and practical for a home lab while introducing key enterprise security concepts such as network segmentation, controlled inter-network communication, centralized identity, authentication, authorization, and security-focused validation.

This project is not intended to replicate a full production enterprise environment. Instead, it provides a controlled foundation where future projects can generate, collect, analyze, and investigate realistic security events.

### Foundation Architecture

The lab is built around three primary network zones:

* **WAN / Internet** — VMware NAT network
* **Users Network** — Dedicated network for user endpoints
* **Server Network** — Dedicated network for infrastructure services

The **pfSense firewall** provides routing and acts as the primary security boundary between these zones.

### Purpose

The main purpose of this foundation is to create an environment that can later support projects involving:

* Network Security Monitoring
* Windows and Endpoint Telemetry
* SIEM and Log Analysis
* Detection Engineering
* MITRE ATT&CK Mapping
* Threat Hunting
* Incident Investigation
* Incident Response
* SOC Automation

## Objectives

The primary objectives of this project are to:

* Build a lightweight, enterprise-style cybersecurity lab suitable for SOC-focused learning and experimentation.
* Establish a segmented network architecture separating user endpoints from server infrastructure.
* Deploy pfSense as the central firewall and security boundary between network segments.
* Establish internal DNS services to support reliable name resolution within the lab.
* Deploy Active Directory to provide centralized identity, authentication, and authorization.
* Connect Windows endpoints to the domain and establish a basic enterprise identity structure.
* Implement controlled communication between network segments using firewall policies.
* Validate network connectivity, segmentation, authentication, authorization, and security controls.
* Establish a reliable foundation for generating and investigating security telemetry in future SOC projects.
* Document the architecture, implementation decisions, security controls, and validation results in a reproducible manner.

## Architecture

The lab follows a simple, segmented architecture designed around a central firewall and separate user and server networks.

```text
                         Internet
                            │
                            │
                     VMware NAT (VMnet8)
                            │
                            │ WAN
                            ▼
                    ┌───────────────┐
                    │    SOC-FW01   │
                    │    pfSense    │
                    │ Firewall/GW   │
                    └───────┬───────┘
                            │
                  ┌─────────┴─────────┐
                  │                   │
             VMnet1               VMnet2
          USERS NETWORK        SERVER NETWORK
          10.10.10.0/24        10.10.20.0/24
                  │                   │
                  │                   │
             SOC-WIN01            SOC-DC01
             Windows Client       Windows Server
                                  AD + DNS
```

### Core Components

| Component     | Role                                                              |
| ------------- | ----------------------------------------------------------------- |
| **SOC-FW01**  | pfSense firewall, gateway, routing, and network security boundary |
| **SOC-WIN01** | Windows endpoint representing a user workstation                  |
| **SOC-DC01**  | Windows Server providing Active Directory and internal DNS        |
| **VMnet8**    | WAN connectivity through VMware NAT                               |
| **VMnet1**    | Users network                                                     |
| **VMnet2**    | Server network                                                    |

### Network Zones

| Zone        | Network           | Purpose                                  |
| ----------- | ----------------- | ---------------------------------------- |
| **WAN**     | `192.168.46.0/24` | External connectivity through VMware NAT |
| **USERS**   | `10.10.10.0/24`   | User endpoints and workstation traffic   |
| **SERVERS** | `10.10.20.0/24`   | Infrastructure services and servers      |

The architecture intentionally uses separate virtual networks for the Users and Servers zones. This provides network segmentation and establishes a clear security boundary that can be enforced through pfSense firewall policies.

The design avoids unnecessary enterprise complexity such as high availability, multiple domain controllers, advanced routing protocols, and large-scale network infrastructure. The goal is to maintain a lightweight environment while preserving the security concepts required for future SOC-focused projects.


## Network Design

The lab network is divided into separate virtual network segments to establish a clear security boundary between external connectivity, user endpoints, and server infrastructure.

### Network Addressing

| Network     | VMware Network | Subnet            | Purpose                                     |
| ----------- | -------------- | ----------------- | ------------------------------------------- |
| **WAN**     | VMnet8         | `192.168.46.0/24` | External connectivity through VMware NAT    |
| **USERS**   | VMnet1         | `10.10.10.0/24`   | User endpoints and workstation traffic      |
| **SERVERS** | VMnet2         | `10.10.20.0/24`   | Server infrastructure and internal services |

### Network Flow

```text
Internet
   │
   ▼
VMware NAT (VMnet8)
   │
   ▼
pfSense WAN
   │
   ├──────────────► USERS
   │                10.10.10.0/24
   │
   └──────────────► SERVERS
                    10.10.20.0/24
```

### Segmentation Approach

The Users and Servers networks are implemented as separate VMware virtual networks and IP subnets.

pfSense will provide the Layer 3 routing between these networks and enforce security policies controlling which traffic is permitted between them.

This approach provides practical network segmentation without introducing unnecessary switching or VLAN infrastructure into the home lab.

### Design Principles

* Separate user and server traffic into distinct network segments.
* Use pfSense as the controlled routing point between internal networks.
* Keep the internal addressing scheme independent from the VMware NAT subnet.
* Avoid unnecessary network complexity while maintaining realistic security boundaries.
* Design the network to support future SOC monitoring and investigation activities.

## Security Controls

The lab uses multiple security controls to establish basic defense-in-depth and controlled communication between network segments.

### Network Security

* **pfSense Firewall** acts as the primary security boundary and routing point between the WAN, Users, and Servers networks.
* **Network Segmentation** separates user endpoints from server infrastructure.
* **Firewall Policies** will control permitted and denied traffic between network segments.
* **Default-deny principles** will be applied where appropriate to minimize unnecessary network access.

### Identity and Access Control

* **Active Directory** provides centralized identity and authentication.
* **Security Groups** will be used to organize permissions and access.
* **Least-privilege principles** will be applied to user and administrative access.
* **Basic Group Policy** will be used where appropriate to establish consistent security settings.

### Security Validation

Security controls will be validated through controlled testing, including:

* Connectivity testing between network segments.
* Verification of permitted and denied traffic.
* DNS resolution testing.
* Domain authentication testing.
* Authorization and access-control validation.

The security controls are intentionally kept lightweight and focused on the requirements of a SOC-oriented home lab. Their primary purpose is to create realistic security boundaries and generate meaningful telemetry for future monitoring and investigation projects.


## Lab Components

The lab consists of a small set of virtual machines and virtual networks, each serving a specific role within the security architecture.

| Component     | Type              | Role                                                      |
| ------------- | ----------------- | --------------------------------------------------------- |
| **SOC-FW01**  | pfSense VM        | Firewall, gateway, routing, and network security boundary |
| **SOC-DC01**  | Windows Server VM | Active Directory Domain Controller and internal DNS       |
| **SOC-WIN01** | Windows Client VM | User endpoint and domain-joined workstation               |
| **VMnet8**    | VMware NAT        | WAN / external connectivity                               |
| **VMnet1**    | VMware Host-only  | Users network                                             |
| **VMnet2**    | VMware Host-only  | Servers network                                           |

### Component Responsibilities

**SOC-FW01**

Provides controlled routing between the WAN, Users, and Servers networks and enforces network security policies.

**SOC-DC01**

Provides centralized identity through Active Directory and internal name resolution through DNS.

**SOC-WIN01**

Represents a typical enterprise user workstation and will provide an endpoint environment for future security monitoring and investigation activities.

**VMware Virtual Networks**

Provide the underlying network separation required to isolate the Users and Servers segments while keeping the lab lightweight and manageable.

> Additional components such as a SIEM server, attack/testing host, endpoint telemetry, and network monitoring tools may be introduced in future projects when they are required.

## Validation

Each major component of the lab will be validated after implementation to ensure that the intended architecture and security controls are functioning correctly.

Validation will cover the following areas:

| Area                     | Validation                                                                |
| ------------------------ | ------------------------------------------------------------------------- |
| **Network Connectivity** | Verify connectivity within and between the required network segments      |
| **Network Segmentation** | Verify that Users and Servers networks are logically separated            |
| **Firewall**             | Verify that permitted traffic is allowed and restricted traffic is denied |
| **DNS**                  | Verify internal name resolution and DNS functionality                     |
| **Active Directory**     | Verify domain functionality, authentication, and basic authorization      |
| **Security Controls**    | Verify that implemented access-control policies behave as intended        |

Validation results and supporting evidence will be documented throughout the project rather than only at the end.

The final validation will confirm that the lab provides a stable and controlled foundation for future SOC-focused projects.

## SOC Relevance

This foundation provides the infrastructure and security boundaries required for future SOC operations and security investigations.

### Network Visibility

The segmented network architecture creates clear security boundaries between users, servers, and external connectivity. This allows future projects to analyze traffic patterns, firewall activity, scanning, lateral movement, and unauthorized access attempts.

### Identity and Authentication Visibility

Active Directory establishes centralized identity and authentication within the lab. This provides a foundation for investigating events such as successful and failed logons, privileged activity, account changes, group membership changes, and other identity-related security events.

### Endpoint Visibility

The Windows endpoint provides a realistic workstation environment that can later generate security telemetry related to processes, network connections, authentication, and user activity.

### Security Monitoring Foundation

The firewall, network segmentation, DNS, Active Directory, and Windows endpoint collectively provide multiple sources of security-relevant telemetry.

These components will later support projects focused on:

* SIEM and centralized log analysis
* Network Security Monitoring
* Endpoint Security Telemetry
* Detection Engineering
* Threat Hunting
* MITRE ATT&CK-based detection
* Incident Investigation
* Incident Response

The foundation therefore acts as the underlying environment in which future SOC projects can generate realistic activity, collect security telemetry, and investigate simulated security incidents.


