# Secure Government Office Network Lab 🔐

A hands-on network security project designed and implemented using Cisco Packet Tracer.

The project simulates a small government office network and applies network segmentation, secure inter-VLAN communication, and access control to protect internal resources.

## 📌 Project Status

🚧 Work in Progress

The network is being developed incrementally, with additional security controls and documentation planned.

## 🏢 Network Scenario

The simulated government office contains three separate network segments:

| VLAN | Department | Network |
|------|------------|---------|
| VLAN 10 | Administration | 192.168.10.0/24 |
| VLAN 20 | IT / Security | 192.168.20.0/24 |
| VLAN 50 | Internal Servers | 192.168.50.0/24 |

Each department is placed in a separate VLAN to reduce unnecessary communication between network segments.

## 🌐 Network Architecture

The current network includes:

- Cisco 1941 Router
- Cisco 2960 Switch
- Administration workstation
- IT/Security workstation
- Internal server

Inter-VLAN communication is implemented using Router-on-a-Stick and an IEEE 802.1Q trunk between the router and switch.

## 🔒 Security Controls Implemented

### VLAN Segmentation

Separate VLANs are used for Administration, IT/Security, and Internal Servers.

### Extended Access Control List (ACL)

An extended ACL named `ADMIN-RESTRICTIONS` is configured to prevent the Administration network from directly accessing the Internal Servers network.

Current access policy:

| Source | Destination | Result |
|--------|-------------|--------|
| Administration | Internal Servers | ❌ Denied |
| Administration | IT / Security | ✅ Allowed |
| IT / Security | Internal Servers | ✅ Allowed |

The ACL follows the principle of least privilege by restricting unnecessary access to sensitive internal resources.

## 🧪 Testing

Connectivity testing was performed using ICMP ping.

Before applying the ACL, the Administration workstation could communicate with the Internal Server.

After applying the ACL:

- Administration → Internal Server: Blocked
- Administration → IT/Security: Successful
- IT/Security → Internal Server: Successful

ACL hit counters were also verified on the router to confirm that the deny rule was actively filtering traffic.

## 🛠️ Technologies

- Cisco Packet Tracer
- Cisco IOS
- VLANs
- IEEE 802.1Q Trunking
- Router-on-a-Stick
- Inter-VLAN Routing
- Extended ACLs

## 📁 Repository Structure

Additional configuration files, screenshots, and the Packet Tracer topology will be added as the project develops.

## 🚀 Planned Improvements

Additional network security controls and documentation will be implemented in future stages of the project.
