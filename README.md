# CyphoraGrid — Enterprise Network Security, Segmentation & Attack Simulation Lab

CyphoraGrid is an enterprise-style network security lab built using Cisco Packet Tracer.

The project focuses on designing a segmented enterprise network and implementing multiple security controls to protect internal departments, servers, management infrastructure and a DMZ environment.

The lab also includes controlled attack simulations, security validation, incident response and network resilience testing.

---

## Project Overview

The objective of CyphoraGrid is to demonstrate how an enterprise network can be designed with security built into the network architecture.

The project includes:

- Enterprise VLAN segmentation
- Inter-VLAN routing
- Router-on-a-Stick architecture
- Access Control Lists (ACLs)
- Secure SSH management
- Port Security
- Sticky MAC address learning
- PortFast and BPDU Guard
- Native VLAN security
- Unused-port isolation
- DMZ segmentation
- STP redundancy
- Link-failure recovery
- Controlled attack simulation
- Incident detection and containment
- Syslog transport
- Security validation and documentation

---

## Architecture

The network consists of:

- 1 Edge Router
- 1 Core Switch
- 2 Access Switches
- Department endpoints
- Internal servers
- DMZ server
- Attacker workstation

### Main Devices

| Device | Role |
|---|---|
| R1-EDGE | Edge router and inter-VLAN routing |
| SW1-CORE | Core/distribution switching |
| SW2-ACCESS | Department access switching |
| SW3-ACCESS | IT Security and DMZ access switching |
| SRV1 | Internal server |
| SRV2 | Syslog server |
| DMZ-SRV1 | DMZ server |
| PC-ATTACKER | Controlled attacker workstation |

---

## VLAN Architecture

| VLAN | Name | Purpose |
|---:|---|---|
| 10 | MANAGEMENT | Management endpoints |
| 20 | FINANCE | Finance department |
| 30 | HR | HR department |
| 40 | IT_SECURITY | IT Security department |
| 50 | SERVERS | Internal servers |
| 60 | DMZ | Public-facing/isolated services |
| 99 | NATIVE_MGMT | Native and management VLAN |
| 999 | BLACKHOLE | Unused/shutdown ports |

---

## IP Addressing

| Network | Gateway | Purpose |
|---|---|---|
| 192.168.10.0/24 | 192.168.10.1 | Management |
| 192.168.20.0/24 | 192.168.20.1 | Finance |
| 192.168.30.0/24 | 192.168.30.1 | HR |
| 192.168.40.0/24 | 192.168.40.1 | IT Security |
| 192.168.50.0/24 | 192.168.50.1 | Servers |
| 192.168.60.0/24 | 192.168.60.1 | DMZ |
| 192.168.99.0/24 | 192.168.99.1 | Management/Native VLAN |

---

## Security Controls

### VLAN Segmentation

Different departments and network functions are separated into dedicated VLANs.

This limits unnecessary Layer-2 communication and creates separate security boundaries within the network.

### Inter-VLAN Routing

R1-EDGE performs routing between VLANs using Router-on-a-Stick.

Each VLAN has a dedicated router subinterface and gateway.

### Access Control Lists

ACLs are used to control traffic between security zones.

The DMZ is restricted from initiating communication toward internal networks.

Traffic from the DMZ toward internal management, department and server networks is restricted.

### SSH Management

Telnet is disabled for device management.

SSH is configured with:

- Local authentication
- SSH version 2
- Restricted VTY access
- Management-source filtering

Management access is permitted from authorized management networks.

### Port Security

Endpoint switch ports use:

- Maximum 1 MAC address
- Sticky MAC learning
- Restrict violation mode

This helps prevent unauthorized devices from being connected to protected access ports.

### Unused Port Security

Unused switch ports are:

- Assigned to VLAN 999
- Administratively shut down

This reduces the attack surface of the switching infrastructure.

### STP Security

PortFast and BPDU Guard are configured on endpoint-facing ports.

STP redundancy is maintained on trunk links.

### DMZ Isolation

The DMZ is separated from internal networks.

The controlled attacker workstation can access permitted DMZ services but is prevented from reaching protected internal networks.

---

## Attack Simulation

The project includes controlled attack simulations using the dedicated attacker workstation.

### Simulated activities

- Internal network reconnaissance
- Gateway probing
- Attempted access to internal server networks
- Unauthorized SSH access attempts
- DMZ-to-internal communication attempts
- Lateral movement attempts

The simulations were performed inside the controlled lab environment.

---

## Incident Response

CyphoraGrid also demonstrates a basic incident-response workflow.

### 1. Detection

Suspicious traffic from the attacker workstation was identified through ACL behavior and connectivity testing.

### 2. Analysis

The source and destination networks were examined to determine which security boundaries were being targeted.

### 3. Containment

The attacker-facing switch port was administratively shut down.

### 4. Recovery

The port was restored after the containment test.

### 5. Validation

Post-incident testing confirmed that:

- DMZ gateway connectivity remained available
- Access to protected internal networks remained blocked
- Security controls remained active

---

## Resilience Testing

A link-failure test was performed to verify network redundancy.

During testing, the connection between SW3-ACCESS and SW1-CORE was temporarily disabled.

The alternate switching path was then used for connectivity.

During this test, a VLAN provisioning issue was identified on SW2-ACCESS. VLANs 40, 50 and 60 were missing from SW2 and were subsequently added.

After correction, connectivity was successfully restored through the alternate path.

Final failover validation achieved:

**4/4 packets — 0% packet loss**

---

## Syslog

Centralized Syslog transport was configured toward:

**SRV2 — 192.168.50.11**

The network devices were configured to send Syslog messages using UDP port 514.

Packet Tracer simulation confirmed the Syslog traffic path between the network devices and SRV2.

The Packet Tracer Syslog service did not populate the displayed Syslog table during testing, so transport verification was used as the validation evidence.

---

## Testing Summary

| Test | Result |
|---|---|
| VLAN segmentation | PASS |
| Inter-VLAN routing | PASS |
| Internal server connectivity | PASS |
| DMZ isolation | PASS |
| Unauthorized internal access | BLOCKED |
| Unauthorized SSH | BLOCKED |
| Port Security | PASS |
| Unused-port lockdown | PASS |
| STP redundancy | PASS |
| Link-failure recovery | PASS |
| Incident containment | PASS |
| Security-control persistence | PASS |
| Final configuration validation | PASS |

---

## Tools & Technologies

- Cisco Packet Tracer
- Cisco IOS
- VLAN
- 802.1Q Trunking
- Inter-VLAN Routing
- ACL
- SSH
- Port Security
- STP / PVST
- Syslog
- Nmap
- Kali Linux
- VMware

---

## Project Structure

```text
CyphoraGrid/
│
├── README.md
│
├── topology/
│   ├── CyphoraGrid.pkt
│   └── network-topology.png
│
├── configurations/
│   ├── R1-EDGE.txt
│   ├── SW1-CORE.txt
│   ├── SW2-ACCESS.txt
│   └── SW3-ACCESS.txt
│
├── documentation/
│   └── CyphoraGrid_Stages.docx
│
├── evidence/
│   ├── dmz-isolation.png
│   ├── Incident Response.png
│   ├── SW1 acl.png
│   ├── SW1 port security.png
│   ├── SW1 trunk.png
│   ├── SW1 vlan.png
│   ├── SW2 acl.png
│   ├── SW2 port security.png
│   ├── SW2 trunk.png
│   ├── SW2 vlan.png
│   ├── SW3 acl.png
│   ├── SW3 port security.png
│   ├── SW3 trunk.png
│   └── SW3 vlan.png
│
└── testing/
    ├── attack-simulation-results.md
    └── validation-results.md
```
