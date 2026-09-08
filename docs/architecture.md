
## Architecture Overview

The **Enterprise SOC Lab Foundation** is designed as a lightweight, segmented enterprise-style environment that provides the infrastructure foundation for future SOC monitoring, detection, investigation, threat hunting, and incident response projects.

The architecture is centered around **SOC-FW01**, a pfSense firewall that acts as the primary network gateway and security boundary between the external network, user endpoints, and server infrastructure.

The internal environment is divided into two separate network segments:

* **Users Network** — `10.10.10.0/24`
* **Servers Network** — `10.10.20.0/24`

The external/WAN connection is provided through **VMware NAT (VMnet8)** using the `192.168.46.0/24` network.

The core infrastructure consists of:

* **SOC-FW01** — pfSense firewall and gateway
* **SOC-WIN01** — Windows client representing a user endpoint
* **SOC-DC01** — Windows Server providing Active Directory and internal DNS
* **VMnet8** — WAN / external connectivity
* **VMnet1** — Users network
* **VMnet2** — Servers network

All communication between the internal network segments is intended to pass through pfSense, allowing security policies to control and restrict inter-network traffic.

The architecture is intentionally kept simple and resource-efficient while maintaining realistic enterprise security concepts and providing a controlled environment for future SOC-focused activities.
