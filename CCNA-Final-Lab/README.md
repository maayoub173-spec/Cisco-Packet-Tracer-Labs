# CCNA Final Project: Enterprise Multi-Branch Network Design & Internet Simulation

Comprehensive technical documentation for a multi-branch enterprise network lab built in **Cisco Packet Tracer**, designed to integrate and apply core **Cisco CCNA** routing, switching, redundancy, and security protocols within a single high-availability architecture.

## 1. Project Overview & Engineering Objective

The objective of this lab is to interconnect two geographically separated sites belonging to the same organization—a **Headquarters (HQ)** and a **Branch Office (BR)**—over a simulated public Internet infrastructure, while providing redundant Internet connectivity and enforcing end-to-end security policies:

* **Headquarters (HQ):** Accommodates **365 employees** across three departments, alongside an on-premises **Data Center** hosting **2 servers**.
* **Branch Office (BR):** Accommodates **80 employees** distributed across four departments.
* **ISP & Simulated Internet Core:** Comprises four ISP routers (dual-homed per site for **Dual-ISP Multihoming**), two core Internet routers, and a central multilayer switch connecting **10 simulated public-service servers** and **one central DNS server**.

| Site | Department | VLAN ID | Planned Capacity (Real-World) | Simulated Endpoints in Lab |
| :--- | :--- | :--- | :--- | :--- |
| **HQ** | Call Center | `VLAN 10` | 200 Employees | 3 Laptops |
| **HQ** | Telecom Engineering | `VLAN 20` | 150 Employees | 3 Laptops |
| **HQ** | Executive | `VLAN 30` | 15 Employees | 3 Laptops |
| **HQ** | Data Center | `VLAN 40` | 2 Servers | 2 Servers (`Server0`, `Server1`) |
| **BR** | Field Engineers | `VLAN 10` | 40 Employees | 2 Laptops |
| **BR** | Regional Sales | `VLAN 20` | 20 Employees | 2 Laptops |
| **BR** | Customer Support | `VLAN 30` | 15 Employees | 2 Laptops |
| **BR** | Branch Management | `VLAN 40` | 5 Employees | 2 Laptops |

## 2. Repository Structure

* **`OVERALL.png`**: Full network topology diagram (HQ, Branch, ISPs, and Simulated Internet).
* **`HQ.png`**: Detailed physical and logical layout of the Headquarters (HQ) network.
* **`BR.png`**: Detailed physical and logical layout of the Branch Office (BR) network.
* **`INTERNET.png`**: Detailed layout of the simulated Internet core, the 10 public-service servers, and the central DNS server.
* **`DNS.png`**: Screenshot of the `A Record` configurations on the central `DNS Server`.

**Topology inventory:** The device descriptions in this README identify **17 network devices** (4 edge routers, 4 ISP routers, 2 Internet core routers, 4 site multilayer switches, 2 access switches, and 1 Internet multilayer switch). The server inventory is separate: 10 simulated public-service servers plus 1 DNS server.

## 3. Physical Topology & 3-Tier Architecture

Both sites follow a symmetric **3-Tier / Collapsed Core Architecture** (Access Layer, Distribution/Core Layer, and Edge/WAN Layer). Redundant links, paired distribution/multilayer switches, dual edge routers, and dual ISP connections are used to improve resilience. This design reduces several potential failure points but does **not** eliminate every single point of failure; for example, each site has one access switch.

### 3.1. Headquarters (HQ) Connections

#### a. Access Layer (`HQ_L2S` – Cisco 2960)

| Port on `HQ_L2S` | Connected Device | Department / Role | Port Configuration |
| :--- | :--- | :--- | :--- |
| `Fa0/1`, `Fa0/2`, `Fa0/3` | 3 Laptops | Call Center (`VLAN 10`) | Access + PortFast + Port Security |
| `Fa0/4`, `Fa0/5`, `Fa0/6` | 3 Laptops | Telecom Engineering (`VLAN 20`) | Access + PortFast + Port Security |
| `Fa0/7`, `Fa0/8`, `Fa0/9` | 3 Laptops | Executive (`VLAN 30`) | Access + PortFast + Port Security |
| `Fa0/10`, `Fa0/11` | 2 Servers (`Server0`, `Server1`) | Data Center (`VLAN 40`) | Access + PortFast + Port Security |
| `Fa0/19`, `Fa0/20`, `Fa0/21` | `HQ_Multi_Layer_Switch_1` (`Fa0/1–3`) | `Port-channel 1` (LACP `active`) | Trunk (802.1Q) + DHCP Snooping Trust |
| `Fa0/22`, `Fa0/23`, `Fa0/24` | `HQ_Multi_Layer_Switch_2` (`Fa0/1–3`) | `Port-channel 2` (LACP `active`) | Trunk (802.1Q) |

#### b. Distribution, Edge, and ISP Layer Connections (HQ)

| Endpoint A (Device & Port) | Endpoint B (Device & Port) | Link Type / Engineering Role |
| :--- | :--- | :--- |
| `HQ_Multi_Layer_Switch_1` (`Fa0/4–6`) | `HQ_Multi_Layer_Switch_2` (`Fa0/4–6`) | `Port-channel 3` (LACP `active` - Trunk 802.1Q) for VLANs & HSRP |
| `HQ_Multi_Layer_Switch_1` (`Fa0/7`) | `HQ_Edge_Router_1` (`Fa0/0`) | Routed Port (`no switchport`) - Subnet `150.150.2.184/30` |
| `HQ_Multi_Layer_Switch_1` (`Fa0/8`) | `HQ_Edge_Router_2` (`Fa0/0`) | Routed Port (`no switchport`) - Subnet `150.150.2.188/30` |
| `HQ_Multi_Layer_Switch_2` (`Fa0/7`) | `HQ_Edge_Router_1` (`Fa0/1`) | Routed Port (`no switchport`) - Subnet `150.150.2.192/30` |
| `HQ_Multi_Layer_Switch_2` (`Fa0/8`) | `HQ_Edge_Router_2` (`Fa0/1`) | Routed Port (`no switchport`) - Subnet `150.150.2.196/30` |
| `HQ_Edge_Router_1` (`Eth0/0/0`) | `HQ_Edge_Router_2` (`Eth0/0/0`) | Direct Inter-Edge Router Link - Subnet `150.150.2.200/30` |
| `HQ_Edge_Router_1` (`Eth0/1/0`) | `HQ_ISP_1` (`Eth0/0/0`) | Primary WAN Link for Edge Router 1 - Subnet `10.0.0.0/30` |
| `HQ_Edge_Router_1` (`Eth0/2/0`) | `HQ_ISP_2` (`Eth0/0/0`) | Secondary WAN Link for Edge Router 1 - Subnet `10.0.0.4/30` |
| `HQ_Edge_Router_2` (`Eth0/1/0`) | `HQ_ISP_1` (`Eth0/1/0`) | Secondary WAN Link for Edge Router 2 - Subnet `10.0.0.8/30` |
| `HQ_Edge_Router_2` (`Eth0/2/0`) | `HQ_ISP_2` (`Eth0/1/0`) | Primary WAN Link for Edge Router 2 - Subnet `10.0.0.12/30` |
| `HQ_ISP_1` (`Eth0/2/0`) | `HQ_ISP_2` (`Eth0/2/0`) | Direct Inter-ISP Link at HQ - Subnet `10.0.0.16/30` |

### 3.2. Branch Office (BR) Connections

#### a. Access Layer (`BR_L2S` – Cisco 2960)

| Port on `BR_L2S` | Connected Device | Department / Role | Port Configuration |
| :--- | :--- | :--- | :--- |
| `Fa0/1`, `Fa0/2` | 2 Laptops | Field Engineers (`VLAN 10`) | Access + PortFast + Port Security |
| `Fa0/3`, `Fa0/4` | 2 Laptops | Regional Sales (`VLAN 20`) | Access + PortFast + Port Security |
| `Fa0/5`, `Fa0/6` | 2 Laptops | Customer Support (`VLAN 30`) | Access + PortFast + Port Security |
| `Fa0/7`, `Fa0/8` | 2 Laptops | Branch Management (`VLAN 40`) | Access + PortFast + Port Security |
| `Fa0/19`, `Fa0/20`, `Fa0/21` | `BR_Multi_Layer_Switch_1` (`Fa0/1–3`) | `Port-channel 1` (PAgP `desirable`) | Trunk (802.1Q) + DHCP Snooping Trust |
| `Fa0/22`, `Fa0/23`, `Fa0/24` | `BR_Multi_Layer_Switch_2` (`Fa0/1–3`) | `Port-channel 2` (PAgP `desirable`) | Trunk (802.1Q) |

#### b. Distribution, Edge, and ISP Layer Connections (BR)

| Endpoint A (Device & Port) | Endpoint B (Device & Port) | Link Type / Engineering Role |
| :--- | :--- | :--- |
| `BR_Multi_Layer_Switch_1` (`Fa0/4–6`) | `BR_Multi_Layer_Switch_2` (`Fa0/4–6`) | `Port-channel 3` (PAgP `desirable` - Trunk 802.1Q) |
| `BR_Multi_Layer_Switch_1` (`Fa0/7`) | `BR_Edge_Router_1` (`Fa0/0`) | Routed Port (`no switchport`) - Subnet `150.150.2.204/30` |
| `BR_Multi_Layer_Switch_1` (`Fa0/8`) | `BR_Edge_Router_2` (`Fa0/0`) | Routed Port (`no switchport`) - Subnet `150.150.2.208/30` |
| `BR_Multi_Layer_Switch_2` (`Fa0/7`) | `BR_Edge_Router_1` (`Fa0/1`) | Routed Port (`no switchport`) - Subnet `150.150.2.212/30` |
| `BR_Multi_Layer_Switch_2` (`Fa0/8`) | `BR_Edge_Router_2` (`Fa0/1`) | Routed Port (`no switchport`) - Subnet `150.150.2.216/30` |
| `BR_Edge_Router_1` (`Eth0/0/0`) | `BR_Edge_Router_2` (`Eth0/0/0`) | Direct Inter-Edge Router Link - Subnet `150.150.2.220/30` |
| `BR_Edge_Router_1` (`Eth0/1/0`) | `BR_ISP_1` (`Eth0/0/0`) | Primary WAN Link for Edge Router 1 - Subnet `10.0.0.48/30` |
| `BR_Edge_Router_1` (`Eth0/2/0`) | `BR_ISP_2` (`Eth0/0/0`) | Secondary WAN Link for Edge Router 1 - Subnet `10.0.0.52/30` |
| `BR_Edge_Router_2` (`Eth0/1/0`) | `BR_ISP_1` (`Eth0/1/0`) | Secondary WAN Link for Edge Router 2 - Subnet `10.0.0.56/30` |
| `BR_Edge_Router_2` (`Eth0/2/0`) | `BR_ISP_2` (`Eth0/1/0`) | Primary WAN Link for Edge Router 2 - Subnet `10.0.0.60/30` |
| `BR_ISP_1` (`Eth0/2/0`) | `BR_ISP_2` (`Eth0/2/0`) | Direct Inter-ISP Link at Branch - Subnet `10.0.0.64/30` |

### 3.3. Simulated Internet Core Connections

| Endpoint A (Device & Port) | Endpoint B (Device & Port) | Assigned Subnet |
| :--- | :--- | :--- |
| `Internet_Router_1` (`Gi0/0`) | `HQ_ISP_1` (`Fa0/0`) | `10.0.0.20/30` |
| `Internet_Router_1` (`Gi1/0`) | `HQ_ISP_2` (`Fa0/0`) | `10.0.0.28/30` |
| `Internet_Router_1` (`Gi2/0`) | `BR_ISP_1` (`Fa0/0`) | `10.0.0.44/30` |
| `Internet_Router_1` (`Gi3/0`) | `BR_ISP_2` (`Fa0/0`) | `10.0.0.40/30` |
| `Internet_Router_1` (`Gi4/0`) | `Internet_Multi_Layer_Switch` (`Gi0/1`) | `10.0.0.36/30` |
| `Internet_Router_2` (`Gi0/0`) | `HQ_ISP_1` (`Fa0/1`) | `10.0.0.24/30` |
| `Internet_Router_2` (`Gi1/0`) | `HQ_ISP_2` (`Fa0/1`) | `10.0.0.32/30` |
| `Internet_Router_2` (`Gi2/0`) | `BR_ISP_1` (`Fa0/1`) | `10.0.0.68/30` |
| `Internet_Router_2` (`Gi3/0`) | `BR_ISP_2` (`Fa0/1`) | `10.0.0.72/30` |
| `Internet_Router_2` (`Gi4/0`) | `Internet_Multi_Layer_Switch` (`Gi0/2`) | `10.0.0.76/30` |

## 4. VLSM & IP Addressing Scheme

### 4.1. Departmental LAN Subnets & HSRP Gateways

The lab address space `150.150.0.0/16` is partitioned using **VLSM** according to departmental host requirements, reserving the first 3 usable addresses in each subnet for the **HSRP Virtual IP** and the two **SVI** addresses on `MLS_1` and `MLS_2`. **Addressing note:** `150.150.0.0/16` is not an RFC 1918 private IPv4 range. It is retained here to document the existing Packet Tracer lab only; do not deploy this addressing plan on a real network or the public Internet. For a real deployment, renumber the internal networks using an RFC 1918 range.

| Site | Department | VLAN ID | Subnet & Prefix | Subnet Mask | HSRP VIP (Default Gateway) | `MLS_1` SVI IP | `MLS_2` SVI IP | Simulated Host / Server IPs |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **HQ** | Call Center | `VLAN 10` | `150.150.0.0/24` | `255.255.255.0` | `150.150.0.1` | `150.150.0.2` | `150.150.0.3` | `.0.4` – `.0.6` (DHCP) |
| **HQ** | Telecom Eng. | `VLAN 20` | `150.150.1.0/24` | `255.255.255.0` | `150.150.1.1` | `150.150.1.2` | `150.150.1.3` | `.1.4` – `.1.6` (DHCP) |
| **HQ** | Executive | `VLAN 30` | `150.150.2.128/27` | `255.255.255.224` | `150.150.2.129` | `150.150.2.130` | `150.150.2.131` | `.2.132` – `.2.134` (DHCP) |
| **HQ** | Data Center | `VLAN 40` | `150.150.2.176/29` | `255.255.255.248` | `150.150.2.177` | `150.150.2.178` | `150.150.2.179` | `.2.180` (`Server0`), `.2.181` (`Server1`) |
| **BR** | Field Engineers | `VLAN 10` | `150.150.2.0/26` | `255.255.255.192` | `150.150.2.1` | `150.150.2.2` | `150.150.2.3` | `.2.4` – `.2.5` (DHCP) |
| **BR** | Regional Sales | `VLAN 20` | `150.150.2.64/27` | `255.255.255.224` | `150.150.2.65` | `150.150.2.66` | `150.150.2.67` | `.2.68` – `.2.69` (DHCP) |
| **BR** | Customer Support | `VLAN 30` | `150.150.2.96/27` | `255.255.255.224` | `150.150.2.97` | `150.150.2.98` | `150.150.2.99` | `.2.100` – `.2.101` (DHCP) |
| **BR** | Branch Mgmt. | `VLAN 40` | `150.150.2.160/28` | `255.255.255.240` | `150.150.2.161` | `150.150.2.162` | `150.150.2.163` | `.2.164` – `.2.165` (DHCP) |

### 4.2. Public Internet Servers & DNS Records

Each simulated public-service server and the central DNS server connect to dedicated access ports and VLANs on `Internet_Multi_Layer_Switch`. The table contains **11 server entries in total: 10 public-service servers and one DNS server**.

**Simulated public-address note:** The table uses documentation ranges such as `203.0.113.0/24`, `198.51.100.0/24`, and `192.0.2.0/24`, but it also contains other example subnets (for example, `203.0.114.0/24` and `198.51.101.0/24`) that are not all reserved documentation ranges. These are lab-only addresses in this simulation and must not be treated as a real public addressing allocation.

| Port on `Internet_MLS` | VLAN ID | Server Name | Server IPv4 Address | Default Gateway (SVI on `Internet_MLS`) | Registered `A Record` on `DNS Server` |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `Fa0/1` | `VLAN 10` | Google Server | `203.0.113.10/24` | `203.0.113.1` | `google.com` |
| `Fa0/2` | `VLAN 20` | Facebook Server | `198.51.100.10/24` | `198.51.100.1` | `facebook.com` |
| `Fa0/3` | `VLAN 30` | YouTube Server | `203.0.114.20/24` | `203.0.114.1` | `youtube.com` |
| `Fa0/4` | `VLAN 40` | Amazon Server | `198.51.101.20/24` | `198.51.101.1` | `amazon.com` |
| `Fa0/5` | `VLAN 50` | Microsoft Server | `203.0.115.30/24` | `203.0.115.1` | `microsoft.com` |
| `Fa0/6` | `VLAN 60` | DNS Server | `192.0.2.53/24` | `192.0.2.1` | Central DNS server (stores the lab's A records) |
| `Fa0/7` | `VLAN 70` | Yahoo Server | `198.51.102.30/24` | `198.51.102.1` | `yahoo.com` |
| `Fa0/8` | `VLAN 80` | Netflix Server | `203.0.116.40/24` | `203.0.116.1` | `netflix.com` |
| `Fa0/9` | `VLAN 90` | Gemini Server | `198.51.103.40/24` | `198.51.103.1` | `gemini.com` |
| `Fa0/10` | `VLAN 100` | Wikipedia Server | `203.0.117.50/24` | `203.0.117.1` | `wikipedia.com` |
| `Fa0/11` | `VLAN 110` | Unwanted Server | `198.51.104.50/24` | `198.51.104.1` | `Unwanted.com` |

## 5. Full-Mesh WAN Redundancy (16 GRE over IPsec Tunnels)

To provide **Full-Mesh WAN Redundancy** across the four edge routers and two ISPs per site—and to transport OSPF multicast packets over encrypted IPsec associations—**16 GRE Tunnels** (`Tunnel1` through `Tunnel16`) are established (4 tunnels per HQ/Branch edge-router pairing):

| Tunnel ID | Subnet (`/30`) | HQ Edge Router & Tunnel IP | HQ Tunnel Source | BR Edge Router & Tunnel IP | BR Tunnel Source | Crypto ACL (HQ / BR) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`Tunnel1`** | `150.150.2.224/30` | `HQ_Edge_1` (`150.150.2.225`) | `Eth0/1/0` (`10.0.0.1`) | `BR_Edge_1` (`150.150.2.226`) | `Eth0/1/0` (`10.0.0.49`) | `ACL 100` / `ACL 100` |
| **`Tunnel2`** | `150.150.2.228/30` | `HQ_Edge_1` (`150.150.2.229`) | `Eth0/1/0` (`10.0.0.1`) | `BR_Edge_1` (`150.150.2.230`) | `Eth0/2/0` (`10.0.0.53`) | `ACL 101` / `ACL 104` |
| **`Tunnel3`** | `150.150.2.232/30` | `HQ_Edge_1` (`150.150.2.233`) | `Eth0/2/0` (`10.0.0.5`) | `BR_Edge_1` (`150.150.2.234`) | `Eth0/1/0` (`10.0.0.49`) | `ACL 104` / `ACL 101` |
| **`Tunnel4`** | `150.150.2.236/30` | `HQ_Edge_1` (`150.150.2.237`) | `Eth0/2/0` (`10.0.0.5`) | `BR_Edge_1` (`150.150.2.238`) | `Eth0/2/0` (`10.0.0.53`) | `ACL 105` / `ACL 105` |
| **`Tunnel5`** | `150.150.2.240/30` | `HQ_Edge_1` (`150.150.2.241`) | `Eth0/1/0` (`10.0.0.1`) | `BR_Edge_2` (`150.150.2.242`) | `Eth0/1/0` (`10.0.0.57`) | `ACL 102` / `ACL 100` |
| **`Tunnel6`** | `150.150.2.244/30` | `HQ_Edge_1` (`150.150.2.245`) | `Eth0/1/0` (`10.0.0.1`) | `BR_Edge_2` (`150.150.2.246`) | `Eth0/2/0` (`10.0.0.61`) | `ACL 103` / `ACL 104` |
| **`Tunnel7`** | `150.150.2.248/30` | `HQ_Edge_1` (`150.150.2.249`) | `Eth0/2/0` (`10.0.0.5`) | `BR_Edge_2` (`150.150.2.250`) | `Eth0/1/0` (`10.0.0.57`) | `ACL 106` / `ACL 101` |
| **`Tunnel8`** | `150.150.2.252/30` | `HQ_Edge_1` (`150.150.2.253`) | `Eth0/2/0` (`10.0.0.5`) | `BR_Edge_2` (`150.150.2.254`) | `Eth0/2/0` (`10.0.0.61`) | `ACL 107` / `ACL 105` |
| **`Tunnel9`** | `150.150.3.0/30` | `HQ_Edge_2` (`150.150.3.1`) | `Eth0/1/0` (`10.0.0.9`) | `BR_Edge_1` (`150.150.3.2`) | `Eth0/1/0` (`10.0.0.49`) | `ACL 100` / `ACL 102` |
| **`Tunnel10`** | `150.150.3.4/30` | `HQ_Edge_2` (`150.150.3.5`) | `Eth0/1/0` (`10.0.0.9`) | `BR_Edge_1` (`150.150.3.6`) | `Eth0/2/0` (`10.0.0.53`) | `ACL 101` / `ACL 106` |
| **`Tunnel11`** | `150.150.3.8/30` | `HQ_Edge_2` (`150.150.3.9`) | `Eth0/2/0` (`10.0.0.13`) | `BR_Edge_1` (`150.150.3.10`) | `Eth0/1/0` (`10.0.0.49`) | `ACL 104` / `ACL 103` |
| **`Tunnel12`** | `150.150.3.12/30` | `HQ_Edge_2` (`150.150.3.13`) | `Eth0/2/0` (`10.0.0.13`) | `BR_Edge_1` (`150.150.3.14`) | `Eth0/2/0` (`10.0.0.53`) | `ACL 105` / `ACL 107` |
| **`Tunnel13`** | `150.150.3.16/30` | `HQ_Edge_2` (`150.150.3.17`) | `Eth0/1/0` (`10.0.0.9`) | `BR_Edge_2` (`150.150.3.18`) | `Eth0/1/0` (`10.0.0.57`) | `ACL 102` / `ACL 102` |
| **`Tunnel14`** | `150.150.3.20/30` | `HQ_Edge_2` (`150.150.3.21`) | `Eth0/1/0` (`10.0.0.9`) | `BR_Edge_2` (`150.150.3.22`) | `Eth0/2/0` (`10.0.0.61`) | `ACL 103` / `ACL 106` |
| **`Tunnel15`** | `150.150.3.24/30` | `HQ_Edge_2` (`150.150.3.25`) | `Eth0/2/0` (`10.0.0.13`) | `BR_Edge_2` (`150.150.3.26`) | `Eth0/1/0` (`10.0.0.57`) | `ACL 106` / `ACL 103` |
| **`Tunnel16`** | `150.150.3.28/30` | `HQ_Edge_2` (`150.150.3.29`) | `Eth0/2/0` (`10.0.0.13`) | `BR_Edge_2` (`150.150.3.30`) | `Eth0/2/0` (`10.0.0.61`) | `ACL 107` / `ACL 107` |

### IPsec Encryption & MTU Tuning

* **`mtu 1476` Configuration:** GRE encapsulation adds 24 bytes of overhead (20-byte outer IPv4 header + 4-byte GRE header). The configured tunnel MTU of `1476` bytes is calculated as $1500 - 24 = 1476$ and accounts for GRE overhead only. **It does not account for the additional IPsec overhead and therefore does not, by itself, guarantee fragmentation-free transmission.** Validate the effective path MTU and TCP MSS for the complete GRE-over-IPsec path.
* **ISAKMP Phase 1:** Policy `10` is configured with `aes` encryption, pre-shared-key authentication, and Diffie-Hellman `Group 2`. The actual pre-shared key is intentionally omitted from this public documentation.
* **IPsec Phase 2:** Transform-set `MY_TS` uses `esp-aes` for payload confidentiality and `esp-sha-hmac` for data integrity, bound to physical WAN interfaces via `crypto map` and triggered by GRE traffic (`permit gre host ...`).

## 6. Implemented Protocols & Technologies

### 6.1. Layer 2 Switching, Aggregation & Access Security

* **VLANs & 802.1Q Trunking:** Departmental isolation into dedicated VLANs, administrative shutdown of unused `Vlan1`, and 802.1Q trunking across all uplink and inter-switch Port-channels.
* **Rapid-PVST+ & PortFast:** Configured `spanning-tree mode rapid-pvst` for sub-second Layer 2 convergence and enabled `spanning-tree portfast` on all access ports connected to laptops and servers.
* **EtherChannel (LACP & PAgP):** Bundled 3 physical FastEthernet links per logical channel to provide bandwidth aggregation and link redundancy, using **LACP (`mode active`)** at HQ and **PAgP (`mode desirable`)** at the Branch.
* **Port Security (`Sticky MAC`):** Enabled on all access interfaces to dynamically learn and save connected MAC addresses into the running configuration and block unauthorized devices.
* **DHCP Snooping:** Configured uplink trunk ports toward `MLS_1` (the DHCP server) as trusted (`ip dhcp snooping trust`) and disabled Option 82 insertion (`no ip dhcp snooping information option`).

### 6.2. Layer 3 Routing, HSRP, DHCP & NAT/PAT

* **Inter-VLAN Routing & HSRP:** Enabled hardware Layer 3 routing (`ip routing`) on `MLS_1` and `MLS_2` with Switched Virtual Interfaces (`SVIs`) per department, paired with **HSRP** (`standby 1` through `standby 4`) for seamless default gateway failover.
* **Centralized DHCP Pools:** Configured `ip dhcp pool` instances on `HQ_Multi_Layer_Switch_1` and `BR_Multi_Layer_Switch_1` to assign host IPs, the HSRP virtual IP as default gateway, and the central DNS server (`192.0.2.53`).
* **Multi-Area OSPF:**
  * **`Area 0` (Backbone):** Runs across all `MLS` switches, `Edge Routers`, and the 16 GRE tunnels (`AD = 110`) to dynamically advertise departmental LAN subnets between HQ and Branch.
  * **`Area 1` (Public WAN/Internet):** Runs across all `ISP` routers, `Internet_Router_1/2`, and `Internet_Multi_Layer_Switch` to route `/30` transit networks and the 11 server `/24` subnets (10 public-service server subnets plus the DNS server subnet).
* **Floating Static Default Routes & Load Balancing:**
  * Default routes (`0.0.0.0/0`) are configured with tiered Administrative Distances (`AD 200` primary, `AD 210` secondary, and `AD 220` tertiary across the inter-edge router link).
  * `MLS_1` prefers `Edge_Router_1` (`AD 200`) which prefers `ISP_1` (`AD 200`), whereas `MLS_2` prefers `Edge_Router_2` (`AD 200`) which prefers `ISP_2` (`AD 200`), achieving load sharing with automatic failover.
* **NAT/PAT Overload & Longest Prefix Match:**
  * Edge routers translate private LAN addresses to public WAN interface addresses via `ip nat inside source list 1 pool <HQ|BR> overload`.
  * When an HQ host communicates with a Branch host, the router matches the specific OSPF route (`/24`, `/26`, `/27`, `/28`, `/29`) learned over the GRE tunnels rather than the `0.0.0.0/0` default route due to **Longest Prefix Match**. Because the `Tunnel1–16` interfaces are not marked as `ip nat outside`, inter-branch packets bypass NAT translation and are encapsulated in GRE and encrypted by IPsec on the physical WAN egress interface.

### 6.3. Network Security Policies, Syslog & Device Hardening

* **Extended Access Control Lists (`Extended ACLs`):**
  * Blocked all HQ and Branch subnets from accessing the `Unwanted Server` (`198.51.104.50`).
  * Blocked web browsing (`TCP Ports 80 & 443`) to the `Wikipedia Server` (`203.0.117.50`) for specific hosts (`150.150.0.5` in HQ and `150.150.2.100` in Branch) while still permitting ICMP `ping`.
  * Restricted access to Data Center `Server0` (`150.150.2.180`) exclusively to the `Telecom Engineering` department (`150.150.1.0/24`).
* **Syslog & Millisecond Timestamps:** Configured all routers and switches to send `debugging`-level logs (`logging trap debugging`) to the central Syslog server (`150.150.2.180`) with `service timestamps log datetime msec`.
* **Administrative Hardening & Backups:** Secured Privileged EXEC mode (`enable secret`), configured local user authentication on `Console` and `VTY` lines, enabled `LLDP` neighbor discovery, and backed up IOS images and configuration files. Usernames, passwords, pre-shared keys, and other credentials should be kept out of public repository files.

## 7. Verification & Testing Commands in Cisco Packet Tracer

Use the following CLI commands on the lab devices to verify operation across each layer:

```ios
! 1. Verify EtherChannel status (LACP at HQ and PAgP at Branch)
show etherchannel summary

! 2. Verify HSRP active/standby gateway election
show standby brief

! 3. Verify OSPF neighbor adjacencies and learned routes over GRE tunnels
show ip ospf neighbor
show ip route ospf

! 4. Verify GRE tunnel state and IPsec Phase 1 / Phase 2 security associations
show ip interface brief | include Tunnel
show crypto isakmp sa
show crypto ipsec sa

! 5. Verify NAT/PAT translations and ACL hit counters
show ip nat translations
show access-lists

! 6. Verify Port Security status and learned Sticky MAC addresses
show port-security
show port-security address
```

## 8. Publication & Lab Scope Notes

* This repository documents a **Cisco Packet Tracer simulation**, not a production-ready network deployment.
* IP addressing, routing behavior, redundancy, and security policies should be validated against the actual `.pkt` topology before being presented as tested results.
* Do not commit real credentials, pre-shared keys, password hashes, private certificates, or sensitive configuration exports to a public repository. Replace any such values in device configuration screenshots or files with placeholders before publishing.
* The design includes redundant paths and devices, but the documented topology still contains potential single points of failure. Avoid describing it as a completely SPOF-free design.
* Tunnel key "pass"
* username Ayoub secret AA55##66
