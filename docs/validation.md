### SERVERS Network Validation

The SERVERS network was successfully validated through the pfSense firewall.

- pfSense SERVERS interface: `10.10.20.1/24`
- Windows host VMnet2 adapter: `10.10.20.2/24`
- ICMP connectivity: Successful
- Packet loss: `0%`
- Average latency: `<1 ms`

This confirms that the VMnet2 network, pfSense `em2` interface, addressing, and required firewall policy are functioning correctly.

### USERS Network Validation

The USERS network was successfully validated through the pfSense LAN interface.

- pfSense LAN interface: `10.10.10.1/24`
- Windows host VMnet1 adapter: `10.10.10.2/24`
- ICMP connectivity: Successful
- Packet loss: `0%`
- Average latency: `0 ms`

This confirms that the VMnet1 network, pfSense `em1` interface, addressing, and LAN connectivity are functioning correctly.

### Active Directory DNS Validation

The Active Directory DNS service was successfully validated on `SOC-DC01`.

- DNS Server: `10.10.20.10`
- Active Directory domain: `corp.local`
- DNS query: Successful
- Resolved address: `10.10.20.10`

The `corp.local` DNS zone correctly resolves to the Domain Controller, confirming that the AD-integrated DNS service is operational.
