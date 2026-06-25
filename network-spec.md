# Network Low-Level Design (LLD) Specifications

This document contains the comprehensive engineering specifications, IP addressing schemes, VLAN architectures, and physical port assignments utilized to build the enterprise network topology.

---

## 1. IP Subnetting & VLAN Allocation Table

The corporate private address space is partitioned using Classless Inter-Domain Routing (CIDR) to enforce strong traffic isolation and optimize broadcast domains.

| VLAN ID | VLAN Name | Subnet Range | Subnet Mask | CS1 SVI IP | CS2 SVI IP | HSRP Virtual IP (Gateway) | Description / Function |
|:---|:---|:---|:---|:---|:---|:---|:---|
| **10** | Data | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.2` | `192.168.10.3` | **`192.168.10.1`** | Corporate End-User PCs |
| **20** | Voice_HR | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.2` | `192.168.20.3` | **`192.168.20.1`** | HR Department & VoIP Infrastructure |
| **99** | Management | `192.168.99.0/24` | `255.255.255.0` | `192.168.99.2` | `192.168.99.3` | *N/A (Direct Access)* | SSH/HTTPS Device Infrastructure Management |

---

## 2. Layer 3 Point-to-Point Interconnections

These routed links use isolated point-to-point subnets to form the routing backbone connecting the Core Switching layer to the Security Edge and the public Internet.

| Link Segment | Connection Diagram | Subnet Range | Subnet Mask | Device A IP | Device B IP |
|:---|:---|:---|:---|:---|:---|
| **CS1 Uplink** | CS1 $\leftrightarrow$ ER | `10.1.1.0/30` | `255.255.255.252` | `10.1.1.2` (CS1) | `10.1.1.1` (ER) |
| **CS2 Uplink** | CS2 $\leftrightarrow$ ER | `10.1.2.0/30` | `255.255.255.252` | `10.1.2.2` (CS2) | `10.1.2.1` (ER) |
| **Public WAN Edge** | ER $\leftrightarrow$ ISP | `203.0.113.0/30` | `255.255.255.252` | `203.0.113.2` (ER) | `203.0.113.1` (ISP) |
| **ISP Server Link** | ISP $\leftrightarrow$ Web Server | `192.51.100.0/24` | `255.255.255.0` | `192.51.100.1` (ISP) | `192.51.100.2` (Server) |

---

## 3. Dynamic Routing & Core Infrastructure Parameters

### OSPF Area 0 Architecture
*   **Process ID:** `router ospf 1`
*   **ER Router ID:** `1.1.1.1`
*   **CS1 Router ID:** `2.2.2.2`
*   **CS2 Router ID:** `3.3.3.3`
*   **Passive Interfaces Engine:** Enforced on `Vlan10`, `Vlan20`, and `Vlan99` to suppress routing advertisements toward access-layer nodes.
*   **Default Route Propagation:** Provisioned on ER using `default-information originate` to dynamically broadcast the internet exit path to the core.

### DHCP Infrastructure Server Pools
Dynamic IP distribution handles automated allocation with the following constraints:
*   **Excluded IP Addresses:** `192.168.10.1` through `192.168.10.5` and `192.168.20.1` through `192.168.20.5` (reserved for network infrastructure nodes).
*   **DNS Node Assignment:** `8.8.8.8` configured globally across all operational subnets.

---

## 4. Physical Port Configuration & Connectivity Map

| Source Device | Source Interface | Cable Type | Destination Device | Destination Interface | Operational Mode / Link Type |
|:---|:---|:---|:---|:---|:---|
| **CS1** | `G1/0/1` | Copper Cross-Over | **CS2** | `G1/0/1` | EtherChannel (`Po1` / LACP Active) |
| **CS1** | `G1/0/2` | Copper Cross-Over | **CS2** | `G1/0/2` | EtherChannel (`Po1` / LACP Active) |
| **CS1** | `G1/0/23` | Copper Cross-Over | **AC1** | `G0/1` | 802.1Q Trunk (Native VLAN 99) |
| **CS1** | `G1/0/24` | Copper Cross-Over | **AC2** | `G0/1` | 802.1Q Trunk (Native VLAN 99) |
| **CS1** | `G1/0/3` | Copper Straight | **ER** | `G0/0/0` | Routed Port (`no switchport`) |
| **CS2** | `G1/0/23` | Copper Cross-Over | **AC1** | `G0/2` | 802.1Q Trunk (Native VLAN 99) |
| **CS2** | `G1/0/24` | Copper Cross-Over | **AC2** | `G0/2` | 802.1Q Trunk (Native VLAN 99) |
| **CS2** | `G1/0/3` | Copper Straight | **ER** | `G0/0/1` | Routed Port (`no switchport`) |
| **AC1** | `F0/1` | Copper Straight | **PC-1** | `FastEthernet0` | Access Port (VLAN 10) |
| **AC1** | `F0/2` | Copper Straight | **PC-2** | `FastEthernet0` | Access Port (VLAN 20) |
| **AC2** | `F0/1` | Copper Straight | **PC-3** | `FastEthernet0` | Access Port (VLAN 10) |
| **AC2** | `F0/2` | Copper Straight | **PC-4** | `FastEthernet0` | Access Port (VLAN 20) |
| **ER** | `G0/0/2` | Copper Cross-Over | **ISP** | `G0/0/0` | Public WAN Boundary |
| **ISP** | `G0/0/1` | Copper Straight | **Web-Server** | `GigabitEthernet0` | Public Server Segment |
