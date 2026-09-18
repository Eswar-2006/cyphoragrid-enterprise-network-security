# 📋 CyphoraGrid: Network Validation & Resilience Test Suite

This document records the baseline verification, inter-VLAN reachability, spanning-tree topology resilience, and port-level security checks conducted across the **CyphoraGrid Enterprise Network**.

---

## 🏗️ Test Environment Summary
- **Network Devices:** 1x Cisco 2911 Edge Router (`R1-EDGE`), 1x Cisco Catalyst 3560 Core Switch (`SW1-CORE`), 2x Cisco Catalyst 2960 Access Switches (`SW2-ACCESS`, `SW3-ACCESS`).
- **L2 Protocol:** PVST+ (Per-VLAN Spanning Tree Plus) with PortFast and BPDU Guard.
- **Trunking:** 802.1Q Encapsulation with dedicated Native VLAN 99 and explicit VLAN pruning (`10,20,30,40,50,60,99`).
- **L3 Protocol:** Inter-VLAN 802.1Q subinterface routing on `R1-EDGE`.

---

## 🧪 Validation Categories & Detailed Results

### 1. Inter-VLAN Routing & Departmental Segmentation
Inter-departmental traffic flows were tested between authorized corporate zones and internal servers.

| Source Node | Source VLAN | Destination Node | Destination VLAN | Protocol | Result | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `PC-MGMT-01` (`.10.10`) | VLAN 10 (Management) | `SRV1` (`.50.10`) | VLAN 50 (Servers) | ICMP / TCP | ✅ SUCCESS | 4/4 packets (0% loss) |
| `PC-FIN-01` (`.20.10`) | VLAN 20 (Finance) | `SRV1` (`.50.10`) | VLAN 50 (Servers) | ICMP / TCP | ✅ SUCCESS | 4/4 packets (0% loss) |
| `PC-HR-01` (`.30.10`) | VLAN 30 (HR) | `SRV1` (`.50.10`) | VLAN 50 (Servers) | ICMP / TCP | ✅ SUCCESS | 4/4 packets (0% loss) |
| `PC-IT-01` (`.40.10`) | VLAN 40 (IT Security) | `SRV1` (`.50.10`) | VLAN 50 (Servers) | ICMP / TCP | ✅ SUCCESS | 4/4 packets (0% loss) |
| `PC-IT-01` (`.40.10`) | VLAN 40 (IT Security) | `SRV2` (`.50.11`) | VLAN 50 (Syslog) | UDP 514 / ICMP | ✅ SUCCESS | Logging operational |

---

### 2. Spanning Tree Protocol (STP) & Link Redundancy Failover
The triangle topology (`SW1` ↔ `SW2` ↔ `SW3`) contains redundant physical links to eliminate single points of failure while preventing broadcast storms and Layer-2 switching loops.

* **Initial State:**
  - `SW1-CORE` functions as Root Bridge for all enterprise VLANs.
  - Active trunks: `SW1-SW2` (`Gi0/1`), `SW1-SW3` (`Gi0/2`), `SW2-SW3` (`Fa0/24`).
  - Redundant link between `SW2` and `SW3` is placed in blocking/alternate state by PVST+.
* **Fault Injection Scenario:**
  - Primary uplink on `SW3-ACCESS` (`Gi0/1` connecting to `SW1-CORE`) was administratively brought down (`shutdown`).
* **Root Cause & Remediated Behavior:**
  - *Observation during test 1:* Initial reroute failed because VLANs 40, 50, and 60 were missing in `SW2-ACCESS`'s local VLAN database despite being trunk-permitted.
  - *Remediation:* Created VLANs 40, 50, 60 in `SW2-ACCESS` database.
  - *Retest Result:* STP recalculated the topology seamlessly. `SW3` redirected frames via `SW2` (`Fa0/24` unblocked).
  - *Ping Test (`PC-IT-01` → `SRV1`):* `4/4 replies (0% packet loss)`.
* **Restoration:**
  - `SW3 Gi0/1` restored (`no shutdown`).
  - STP reconverged to optimal primary forwarding path.

---

### 3. Layer-2 Switchport Security & Hardening

* **Port Security Status on Access Endpoints:**
  - Maximum MAC count: `1`
  - Sticky MAC learning: Active
  - Violation mode: `restrict` (Drops violating frames, increments counter, logs without disabling trunk or critical links).
  - Violation counts across all production edge ports: `0 violations` under normal authorized operation.
* **Unused Port Lockdown:**
  - All unallocated FastEthernet ports on `SW1`, `SW2`, `SW3` assigned to blackhole VLAN `999` and placed in administrative `shutdown`.
  - Native VLAN `1` disabled on all trunks (`shutdown interface Vlan1`).
  - DTP (Dynamic Trunking Protocol) disabled using `switchport nonegotiate` to mitigate VLAN hopping attacks.

---

### 4. Management Plane Hardening (SSHv2 & AAA)

* **SSHv2 Transport:** Enforced exclusively (`transport input ssh`), Telnet disabled.
* **Access-Class Enforcement:**
  - Authorized sources (`192.168.10.0/24`, `192.168.40.0/24`): Successfully initiated SSH session with local admin privilege level 15.
  - Unauthorized sources (`192.168.20.0/24`, `192.168.30.0/24`, `192.168.60.0/24`): Dropped at transport establishment.
* **Password Encryption:** Cisco Type 5 MD5 secret hashes used for `enable secret` and local admin user; Type 7 encryption on console line passwords.

---

## 📈 Final Validation Matrix

| Layer | Security Objective | Target Requirement | Status |
| :--- | :--- | :--- | :--- |
| **L2** | Loop Prevention | PVST+ loop-free topology with automatic redundancy | 🟢 PASS |
| **L2** | Trunk Hardening | 802.1Q tagged, Native VLAN 99, DTP Nonegotiate | 🟢 PASS |
| **L2** | Access Hardening | Sticky Port-Security (Max 1, Restrict), Blackhole VLAN 999 | 🟢 PASS |
| **L3** | Zone Segmentation | Isolated subnets across 6 core departmental VLANs | 🟢 PASS |
| **L3** | DMZ Perimeter Control | `DMZ-IN` Extended ACL restricts lateral hopping | 🟢 PASS |
| **L3** | Server Protection | `SERVER-OUT` Extended ACL filters unauthorized zones | 🟢 PASS |
| **Mgmt** | SSH Hardening | SSHv2, RSA crypto keys, VTY `MGMT-SSH` access-class | 🟢 PASS |
| **Ops** | Central Logging | Syslog trap telemetry directed to `192.168.50.11` | 🟢 PASS |
