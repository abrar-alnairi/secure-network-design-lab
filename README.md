# 🔐 Secure Government Office Network Lab

> **Status: Work in Progress**

A cybersecurity networking lab designed and implemented using Cisco Packet Tracer to simulate a secure multi-site government office network.

The project currently includes a segmented Head Office, a Warehouse branch, a simulated point-to-point WAN connection, and a Site-to-Site IPsec VPN. It demonstrates network segmentation, access control, secure device management, Layer 2 hardening, DMZ isolation, branch connectivity, and encrypted site-to-site communication.

---

## 🎯 Project Objectives

The main objectives of this project are to:

- Segment departments and services using VLANs
- Implement inter-VLAN routing using Router-on-a-Stick
- Restrict unauthorized communication using Access Control Lists (ACLs)
- Protect switch access ports using Port Security
- Secure unused switch ports
- Restrict Head Office router management using SSH and a management ACL
- Separate the DMZ from internal server resources
- Host a simulated HTTPS web service inside the DMZ
- Deploy a Warehouse branch network
- Establish routed connectivity between the Head Office and Warehouse
- Protect inter-site traffic using a Site-to-Site IPsec VPN
- Validate implemented security controls through functional testing

---

## 🏢 Network Scenario

The current implementation represents two sites of a small government organization:

- **Head Office** — Administration, IT/Security, Internal Servers, and DMZ
- **Warehouse Branch** — Dedicated Warehouse network

The two sites are connected through a simulated point-to-point WAN using:

```text
10.0.0.0/30
```

A Site-to-Site IPsec VPN protects selected traffic between the Head Office and Warehouse networks.

### Network Segments

| VLAN | Department / Zone | Network | Default Gateway |
|---|---|---|---|
| 10 | Administration | `192.168.10.0/24` | `192.168.10.1` |
| 20 | IT / Security | `192.168.20.0/24` | `192.168.20.1` |
| 50 | Internal Servers | `192.168.50.0/24` | `192.168.50.1` |
| 60 | DMZ | `192.168.60.0/24` | `192.168.60.1` |
| 70 | Warehouse | `192.168.70.0/24` | `192.168.70.1` |
| 999 | Unused Ports | N/A | N/A |

### WAN Addressing

| Device | Interface Role | IP Address |
|---|---|---|
| `GOV-R1` | Head Office WAN | `10.0.0.1/30` |
| `WH-R1` | Warehouse WAN | `10.0.0.2/30` |

---

## 🖥️ Main Devices

| Device | Role | Address / Network |
|---|---|---|
| `GOV-R1` | Head Office ISR4331 Router | VLAN gateways + WAN + IPsec VPN |
| `GOV-SW1` | Head Office Cisco 2960 Switch | VLAN segmentation |
| `ADMIN-PC` | Administration Workstation | `192.168.10.10/24` |
| `SECURITY-PC` | IT/Security Workstation | `192.168.20.10/24` |
| `INTERNAL-SERVER` | Internal Server | `192.168.50.10/24` |
| `DMZ-WEB-SERVER` | DMZ Web Server | `192.168.60.10/24` |
| `WH-R1` | Warehouse ISR4331 Router | `192.168.70.1/24` + WAN |
| `WH-SW1` | Warehouse Cisco 2960 Switch | Warehouse access network |
| `WAREHOUSE-PC` | Warehouse Workstation | `192.168.70.10/24` |

---

## 🗺️ Network Topology

The Head Office uses an 802.1Q trunk between `GOV-SW1` and `GOV-R1` to carry VLANs 10, 20, 50, and 60.

The Warehouse operates on VLAN 70 and connects to the Head Office through a simulated point-to-point WAN. A Site-to-Site IPsec VPN is configured between `GOV-R1` and `WH-R1`.

![Network Topology](screenshots/network.png)

---

# 🛡️ Implemented Security Controls

## 1. VLAN Segmentation

The network is divided into separate VLANs for:

- Administration
- IT/Security
- Internal Servers
- DMZ
- Warehouse
- Unused switch ports

This provides logical separation between organizational functions and reduces unnecessary Layer 2 broadcast exposure.

---

## 2. Router-on-a-Stick Inter-VLAN Routing

`GOV-R1` provides routing between the Head Office VLANs using 802.1Q subinterfaces:

```text
G0/0/0.10 → VLAN 10
G0/0/0.20 → VLAN 20
G0/0/0.50 → VLAN 50
G0/0/0.60 → VLAN 60
```

The connection between `GOV-R1` and `GOV-SW1` operates as an 802.1Q trunk.

---

## 3. Administration Access Control

An extended ACL named:

```text
ADMIN-RESTRICTIONS
```

prevents the Administration VLAN from accessing the Internal Server VLAN.

Security policy:

```text
ADMIN → Internal Servers = DENIED
ADMIN → Other permitted networks = ALLOWED
```

The policy was validated through connectivity testing.

---

## 4. DMZ Isolation

A dedicated DMZ is implemented using VLAN 60.

The DMZ hosts:

```text
DMZ-WEB-SERVER
192.168.60.10/24
```

An extended ACL named:

```text
DMZ-RESTRICTIONS
```

prevents the DMZ from initiating communication with the Internal Server VLAN.

Current tested policy:

```text
DMZ → Internal Servers = DENIED
DMZ → IT/Security       = ALLOWED
```

This provides logical separation between the DMZ service and internal server resources.

---

## 5. HTTPS Web Service

`DMZ-WEB-SERVER` hosts a simulated Government Office Portal.

The server is configured with:

```text
HTTP  = Disabled
HTTPS = Enabled
```

HTTPS access was successfully tested from the IT/Security workstation.

> HTTPS functionality is demonstrated within the Cisco Packet Tracer simulation environment and should not be interpreted as production-grade TLS validation.

---

## 6. Port Security

Port Security with sticky MAC learning is implemented on active endpoint access ports at both sites.

### Head Office

| Interface | Device | VLAN |
|---|---|---:|
| `Fa0/2` | ADMIN-PC | 10 |
| `Fa0/3` | SECURITY-PC | 20 |
| `Fa0/4` | INTERNAL-SERVER | 50 |
| `Fa0/5` | DMZ-WEB-SERVER | 60 |

An unauthorized-device test was performed on the Administration access port, causing the port to enter a secure-shutdown state after a security violation.

### Warehouse

| Interface | Device | VLAN |
|---|---|---:|
| `Fa0/2` | WAREHOUSE-PC | 70 |

The Warehouse access port uses sticky MAC learning and was verified in a secure operational state.

---

## 7. Unused Port Hardening

Unused switch ports at both the Head Office and Warehouse are:

- Assigned to VLAN 999 (`UNUSED-PORTS`)
- Configured as access ports
- Administratively shut down

This reduces exposure from unused physical switch interfaces.

---

## 8. Secure SSH Management

Remote management of `GOV-R1` uses:

```text
SSH Version 2
```

Telnet is not permitted on its VTY lines.

A local privileged administrative account is used for authentication.

> Authentication secrets are intentionally excluded from the public repository.

---

## 9. SSH Management ACL

A standard ACL named:

```text
SSH-MANAGEMENT
```

restricts remote management of `GOV-R1` to:

```text
192.168.20.0/24
```

Therefore:

```text
SECURITY-PC → SSH → GOV-R1 = ALLOWED
ADMIN-PC    → SSH → GOV-R1 = DENIED
```

Both conditions were functionally tested.

---

## 10. Warehouse Branch Network

A Warehouse branch was deployed using:

```text
VLAN 70
Network: 192.168.70.0/24
Gateway: 192.168.70.1
```

`WH-R1` provides the Warehouse default gateway, while `WH-SW1` provides Layer 2 access for the Warehouse workstation.

Local Warehouse connectivity to the default gateway was successfully verified.

---

## 11. Inter-Site Static Routing

Static routes provide Layer 3 reachability between the Head Office and Warehouse.

`GOV-R1` routes the Warehouse network through:

```text
192.168.70.0/24 → 10.0.0.2
```

`WH-R1` contains routes to the Head Office networks through:

```text
192.168.10.0/24 → 10.0.0.1
192.168.20.0/24 → 10.0.0.1
192.168.50.0/24 → 10.0.0.1
192.168.60.0/24 → 10.0.0.1
```

---

## 12. Site-to-Site IPsec VPN

A Site-to-Site IPsec VPN is implemented between:

```text
GOV-R1: 10.0.0.1
WH-R1:  10.0.0.2
```

The VPN protects selected traffic between the Warehouse network and the Head Office VLANs.

The implementation includes:

- IKE/ISAKMP policy
- Pre-shared-key authentication
- IPsec transform set
- Crypto ACLs defining interesting traffic
- Crypto maps applied to both WAN interfaces

Sensitive VPN authentication material is intentionally excluded from the public configuration files.

> The cryptographic algorithms available in this lab are constrained by Cisco Packet Tracer. They are used to demonstrate IPsec concepts and configuration workflow and should not be interpreted as current production cryptographic recommendations.

---

## 13. VPN Verification

The Site-to-Site VPN was functionally verified using traffic between:

```text
WAREHOUSE-PC → SECURITY-PC
192.168.70.10 → 192.168.20.10
```

Successful verification included:

```text
show crypto isakmp sa
```

with an active IKE security association, and:

```text
show crypto ipsec sa
```

showing IPsec packet encryption activity.

This confirms that matching inter-site traffic successfully triggered the VPN tunnel in the Packet Tracer environment.

---

# 🧪 Security Testing

The implemented controls were validated using multiple functional tests, including:

- Administration-to-Internal-Server access denial
- IT/Security-to-Internal-Server access success
- DMZ-to-Internal-Server access denial
- DMZ-to-IT/Security access success
- Authorized SSH management
- Unauthorized SSH management denial
- Head Office Port Security violation detection
- DMZ Port Security verification
- Head Office unused port hardening verification
- HTTPS service accessibility
- HTTP service disablement
- Warehouse local gateway connectivity
- Warehouse Port Security verification
- Warehouse unused port hardening verification
- Warehouse-to-Head-Office connectivity
- IKE security association verification
- IPsec encryption verification

Testing evidence is documented in:

[`screenshots/`](screenshots/)

---

## 📁 Repository Structure

```text
secure-network-design-lab/
│
├── README.md
│
├── configs/
│   ├── GOV-R1-config.txt
│   ├── GOV-SW1-config.txt
│   ├── WH-R1-config.txt
│   └── WH-SW1-config.txt
│
├── packet-tracer/
│   └── [Packet Tracer lab files]
│
└── screenshots/
    ├── README.md
    ├── network.png
    ├── acl-admin-server-denied.png
    ├── acl-security-server-allowed.png
    ├── ssh-security-pc-success.png
    ├── ssh-admin-pc-denied.png
    ├── port-security-violation.png
    ├── unused-ports-hardening.png
    ├── dmz-internal-server-denied.png
    ├── dmz-security-pc-allowed.png
    ├── dmz-port-security.png
    ├── dmz-https-success.png
    ├── dmz-http-disabled.png
    ├── warehouse-gateway-connectivity.png
    ├── warehouse-port-security.png
    ├── warehouse-unused-ports-hardening.png
    ├── vpn-warehouse-to-security-connectivity.png
    ├── vpn-isakmp-sa-active.png
    └── vpn-ipsec-encryption-evidence.png
```

---

## 🧰 Technologies & Concepts

- Cisco Packet Tracer
- Cisco IOS
- Cisco ISR4331
- Cisco 2960 Switches
- VLANs
- IEEE 802.1Q Trunking
- Router-on-a-Stick
- Inter-VLAN Routing
- Static Routing
- Access Control Lists (ACLs)
- DMZ Segmentation
- Port Security
- Sticky MAC Learning
- SSH Version 2
- Layer 2 Hardening
- HTTPS Service Simulation
- Point-to-Point WAN Simulation
- Site-to-Site IPsec VPN
- IKE / ISAKMP
- IPsec Security Associations
- Network Security Testing

---

## 🚀 Planned Improvements

The project will continue to evolve through additional security hardening and validation.

Potential future improvements include:

- Additional Warehouse traffic filtering and access-control policies
- Enhanced secure management of branch infrastructure
- Expanded monitoring and logging
- Additional security validation and attack-simulation scenarios
- Further documentation and architecture refinement

---

## ⚠️ Lab Scope

This project is implemented in Cisco Packet Tracer as an educational cybersecurity lab.

The current architecture includes a **simulated point-to-point WAN connection** between the Head Office and Warehouse. This WAN link represents inter-site connectivity inside the lab and does not represent a real ISP or public Internet connection.

The project currently does **not** implement:

- Public Internet connectivity
- ISP infrastructure
- NAT
- A perimeter firewall
- Production-grade VPN cryptography
- Enterprise monitoring infrastructure

The DMZ represents a segmented server zone within the simulated enterprise environment.

The project should therefore not be interpreted as a complete production government network architecture.

---

## 👩‍💻 Author

**Abrar Al-Nairi**

Computer Engineering — Cybersecurity  
Middle East College, Oman

---

## 📌 Project Status

**Work in Progress**

Current implementation:

**Head Office VLAN Segmentation + Inter-VLAN Routing + ACLs + Port Security + Unused Port Hardening + SSH Security + DMZ Isolation + HTTPS Web Service + Warehouse Branch + Static Routing + Site-to-Site IPsec VPN**

Next phase:

**Security Hardening & Validation**
