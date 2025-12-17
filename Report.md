##  Project  Overview

<img width="2543" height="1053" alt="image" src="https://github.com/user-attachments/assets/15fbce59-e68b-4727-b01b-afa08ecd8c09" />


# University Campus Network Design Project

**Project Scope:** Full connectivity for 4 Buildings (CS, Engineering, Admin, Library) to the Internet.
**Architecture:** Hierarchical Design with Enterprise-Grade Security.

---

## 1️⃣ Rubric Requirements :

We have strictly adhered to the project rubric while implementing industry-standard enhancements.

| **Criteria** | **Status** | **Implementation Details** |
| --- | --- | --- |
| **DHCP** | ✅ **Done** | Configured on Building Routers to automatically assign IPs, Gateways, and DNS (`8.8.8.8`) to all PCs. |
| **Sub-netting** | ✅ **Done** | Efficient VLSM addressing used for all 4 buildings (`10.0.0.0/8` block) to separate Labs, Faculty, and Admin. |
| **ACL** | ✅ **Done** | Implemented **Access Control Lists** on routers to BLOCK Students from accessing Faculty networks while allowing Internet access. |
| **Routing** | ✅ **Done** | **OSPF Area 0** configured on all routers for full internal connectivity + **BGP** for ISP connection. |
| **SSH** | ✅ **Done** | Telnet disabled. **SSH v2** configured with RSA encryption (1024-bit) on all devices for secure remote management. |

---

## 2️⃣ Bonus & Advanced Features 

Beyond the basic requirements, we implemented **5 Advanced Features** to simulate a real-world ISP and Enterprise environment:

1. **Real-World ISP Simulation (BGP):**
* Instead of a simple static route, we configured **BGP (Border Gateway Protocol)** between the Main Router and the ISP Router.
* Simulates true internet routing between Autonomous Systems (AS 65000 & AS 65001).


2. **NAT Overload (Port Address Translation):**
* The rubric asks for routing, but in reality, private IPs (`10.x.x.x`) cannot route on the internet.
* We implemented **NAT Overload** on the Main Router to translate thousands of internal private IPs to a single Public IP (`200.1.1.1`).


3. **Layer 2 Security (Port Security):**
* Implemented **Sticky MAC Learning** on Access Switches.
* **Policy:** Faculty ports utilize `Shutdown` mode (Zero Tolerance) against rogue devices, while Student ports use `Restrict` mode.


4. **Network Isolation (Management VLAN):**
* Created a dedicated **VLAN 99** for network administrators, isolating management traffic (SSH) from user data traffic to ensure stability.


5. **Performance Optimization (Rapid-PVST+):**
* Upgraded Spanning Tree Protocol to **Rapid-PVST+** and enabled `PortFast`.
* Ensures ports become active in sub-seconds (vs. 30s standard), preventing DHCP timeouts.



---

## 3️⃣ Network Topology Overview

The network connects **four main buildings** via a central backbone:

1. **Computer Science (CS)**
2. **Engineering (ENG)**
3. **Administration (Admin)**
4. **Library (Lib)**

All buildings connect to a **Main Campus Router**, which acts as the Gateway to the **ISP Router** (Internet).

---

## 4️⃣ Core Network Configuration (Backbone)

**Verified from Device Configuration:**

### A. The Internet Service Provider (ISP)

* **Role:** Simulates the Global Internet.
* **Target:** `Loopback0` interface configured as `8.8.8.8` (Public DNS) for connectivity testing.
* **Routing:** Uses BGP to exchange routes with the University.

### B. Main Campus Router (Edge Gateway)

This is the most critical device, handling all ingress/egress traffic.

* **NAT Configuration:**
`ip nat inside source list 1 interface Serial0/0/0 overload`
*(Maps all 4 buildings' traffic to the WAN IP).*
* **Hybrid Routing:**
* **OSPF:** Redistributes the default route to all 4 buildings so they know how to reach the internet.
* **BGP:** Advertises the university's public network to the ISP.



---

## 5️⃣ Building Implementation (CS, Eng, Admin, Lib)

Each building follows a standard **Hierarchical Model** (Access & Distribution Layers) with unique subnets.

### A. VLAN Strategy (Segmentation)

We replaced flat networks with segmented VLANs for security:

* **VLAN 10/20/30/40:** Labs (Students/Public).
* **VLAN 11/21/31/41:** Faculty/Staff (Secured).
* **VLAN 99:** Management (Admins only).

### B. Router-on-a-Stick (ROAS)

Building routers (CS-Router, Eng-Router, etc.) use sub-interfaces (`G0/0.10`, `G0/0.20`) to route traffic between these VLANs efficiently using a single physical cable.

### C. Access Layer Security

Access switches (Cisco 2960) are hardened:

* **Unused Ports:** Shutdown.
* **Active Ports:** Configured with `switchport port-security` to prevent "Man-in-the-Middle" attacks.

---

## 6️⃣ Verification Results (Proof of Work)

| Test Scenario | Source | Destination | Result | Feature Validated |
| --- | --- | --- | --- | --- |
| **Internet Access** | **Library PC** | **8.8.8.8 (Google)** | ✅ **Success** | NAT & BGP |
| **Campus Routing** | **Admin PC** | **CS Lab PC** | ✅ **Success** | OSPF Area 0 |
| **Security Rule** | **CS Student** | **CS Faculty** | ✅ **Blocked** | ACL (Access List) |
| **Rogue Device** | **Hacker Laptop** | **Switch Port** | ✅ **Port Down** | Port Security |
| **Auto-Config** | **Any PC** | **-** | ✅ **IP Received** | DHCP Server |

---

### **Conclusion**

This project delivers a **secure, scalable, and realistic** enterprise network. By fulfilling all rubric requirements and adding advanced features like **NAT, BGP, and Port Security**, the design simulates a true production environment ready for deployment.
