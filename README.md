# **Project Report: Technical College Campus Network**

<img width="2543" height="1053" alt="image" src="https://github.com/user-attachments/assets/15fbce59-e68b-4727-b01b-afa08ecd8c09" />

## **Project Overview**

This project aims to design and simulate a scalable, secure, and robust network infrastructure for a **Technical College Campus**. The network connects four distinct buildings—**Computer Science (CS), Engineering, Administration, and Library**—to a central Core Router, facilitating high-speed internal communication and external Internet access via an ISP connection.

**Topology Architecture:**
The network utilizes a **Hierarchical Extended Star Topology**. The **Main Campus Router** acts as the central backbone (Core Layer), connecting to the four building routers in a **Hub-and-Spoke** configuration via high-speed serial links. Within each building, a **Router-on-a-Stick** design connects to Distribution switches, which then branch out to Access switches, ensuring modularity and ease of troubleshooting.

---

## **1. Executive Summary**

The design implements a segmented network architecture using **VLANs** for departmental isolation, **OSPF** for dynamic internal routing, and **BGP** for external connectivity. Robust security measures, including **Access Control Lists (ACLs)**, **Port Security**, and **SSH**, are enforced to protect sensitive data and restrict unauthorized access between students, faculty, and administrative staff.

---

## **2. Technologies & Protocols Implemented**

The following technologies were deployed to meet the design requirements:

1. **VLSM (Variable Length Subnet Masking):** Optimized IP allocation based on VLAN density.
2. **DHCP (Dynamic Host Configuration Protocol):** Automated IP assignment with specific exclusions for gateways and servers.
3. **SSH (Secure Shell v2):** Encrypted remote management for all network devices (Telnet disabled).
4. **VLANs (Virtual Local Area Networks):** Logical segmentation of traffic (Student, Staff, Admin, Management).
5. **Inter-VLAN Routing (Router-on-a-Stick):** Enabled communication between VLANs via 802.1Q encapsulation.
6. **OSPF (Open Shortest Path First):** Single-area dynamic routing for internal campus connectivity.
7. **BGP (Border Gateway Protocol):** Peering with the ISP for internet simulation.
8. **ACL (Access Control Lists):** Traffic filtering policies to secure sensitive departments.
9. **Port Security:** Layer 2 protection using Sticky MAC addresses and violation actions.
10. **NAT (Network Address Translation):** Overload (PAT) configuration for internet access.

---

## **3. Network Topology & Addressing Plan**

### **A. WAN Backbone (Inter-Router Links)**

| Link | Network Address | Subnet Mask | Protocol |
| --- | --- | --- | --- |
| Main ↔ CS | `10.0.10.0` | `/30` | OSPF Area 0 |
| Main ↔ Engineering | `10.0.10.4` | `/30` | OSPF Area 0 |
| Main ↔ Admin | `10.0.10.8` | `/30` | OSPF Area 0 |
| Main ↔ Library | `10.0.10.12` | `/30` | OSPF Area 0 |
| Main ↔ ISP | `200.1.1.0` | `/30` | BGP AS 65000 |

### **B. Building-Level IP Allocation (VLSM Implementation)**

*Based on actual device configurations.*

#### **1. Computer Science (CS) Building**

| VLAN | Name | Network ID | Subnet Mask | Gateway |
| --- | --- | --- | --- | --- |
| **10** | CS-LABS | `10.0.0.0` | `/25` (255.255.255.128) | `10.0.0.1` |
| **11** | CS-FACULTY | `10.0.0.128` | `/26` (255.255.255.192) | `10.0.0.129` |
| **12** | CS-TAS | `10.0.0.192` | `/26` (255.255.255.192) | `10.0.0.193` |
| **99** | MGMT-CS | `10.0.254.0` | `/26` (255.255.255.192) | `10.0.254.1` |

#### **2. Engineering (ENG) Building**

| VLAN | Name | Network ID | Subnet Mask | Gateway |
| --- | --- | --- | --- | --- |
| **20** | ENG-LABS | `10.0.1.0` | `/26` (255.255.255.192) | `10.0.1.1` |
| **21** | ENG-FACULTY | `10.0.1.64` | `/27` (255.255.255.224) | `10.0.1.65` |
| **22** | ENG-ADMIN | `10.0.1.96` | `/27` (255.255.255.224) | `10.0.1.97` |
| **99** | MGMT-ENG | `10.0.254.64` | `/26` (255.255.255.192) | `10.0.254.65` |

#### **3. Administration (ADMIN) Building**

| VLAN | Name | Network ID | Subnet Mask | Gateway |
| --- | --- | --- | --- | --- |
| **30** | ADMIN-STAFF | `10.0.1.128` | `/26` (255.255.255.192) | `10.0.1.129` |
| **31** | ADMIN-STUDENTS | `10.0.1.192` | `/27` (255.255.255.224) | `10.0.1.193` |
| **32** | ADMIN-ACCOUNTS | `10.0.1.224` | `/28` (255.255.255.240) | `10.0.1.225` |
| **33** | ADMIN-SERVICES | `10.0.1.240` | `/28` (255.255.255.240) | `10.0.1.241` |
| **99** | MGMT-ADMIN | `10.0.254.128` | `/26` (255.255.255.192) | `10.0.254.129` |

#### **4. Library (LIB) Building**

| VLAN | Name | Network ID | Subnet Mask | Gateway |
| --- | --- | --- | --- | --- |
| **40** | LIB-STUDENTS | `10.0.2.0` | `/26` (255.255.255.192) | `10.0.2.1` |
| **41** | LIB-STAFF | `10.0.2.64` | `/28` (255.255.255.240) | `10.0.2.65` |
| **42** | LIB-SERVERS | `10.0.2.80` | `/28` (255.255.255.240) | `10.0.2.81` |
| **43** | LIB-PRINTERS | `10.0.2.96` | `/28` (255.255.255.240) | `10.0.2.97` |
| **99** | MGMT-LIB | `10.0.254.192` | `/26` (255.255.255.192) | `10.0.254.193` |

---

## **4. Configuration Details & Security Implementation**

### **A. Dynamic Host Configuration Protocol (DHCP)**

DHCP services are centralized on the local router for each building.

* **Excluded Addresses:** The first 10 IPs of each subnet are reserved for Gateways and Static allocation to prevent conflicts.
* **DNS Configuration:** All pools are configured to use `8.8.8.8`.
* **Implementation:** Separate pools created for every VLAN (e.g., `CS-LABS`, `ENG-FACULTY`).

### **B. Routing Configuration**

* **Internal (OSPF):** All building routers and the Main router are part of **Area 0**. Network advertisements allow full mesh connectivity between all subnets.
* **External (BGP & NAT):** The Main Router peers with the ISP (AS 65001). A **NAT Overload** rule permits internal traffic (Access-List 1) to access the internet via the public IP `200.1.1.1`.

### **C. Access Control Lists (ACLs)**

Security policies were implemented to enforce departmental segregation.

* **Policy:** Students/Labs are denied access to Faculty and Admin networks.
* **Implementation:** Standard ACL `100` applied **Inbound** on Lab Gateways.
* `deny ip [Lab_Subnet] [Faculty_Subnet]`
* `deny ip [Lab_Subnet] [Admin_Subnet]`
* `permit ip any any` (Allows Internet & other traffic).



### **D. SSH & Management Security**

Remote management is secured across all devices:

* **Protocol:** SSH version 2 enabled.
* **Encryption:** RSA Keys (1024-bit) generated.
* **Authentication:** Local Username/Password (`admin`/`Admin@123`) with Privilege 15.
* **Isolation:** A dedicated **VLAN 99** is used for Switch management IP addresses, separating management traffic from user data.

### **E. Layer 2 Security (Switching)**

* **Port Security:**
* **Labs:** Max 2 MAC addresses, Violation Mode `Restrict`.
* **Faculty/Admin:** Max 1 MAC address, Violation Mode `Shutdown`.
* **Sticky MAC:** Enabled to learn connected devices dynamically.


* **Performance:** `spanning-tree portfast` enabled on all access ports to ensure immediate connectivity for DHCP clients.

---

## **5. Conclusion**

The implemented network successfully meets the connectivity requirements for the Technical College. It provides a segmented, routed environment with automated IP assignment and strict security controls compliant with the project rubric. The design ensures that Faculty and Admin resources are protected while providing students with necessary resource access.

This project delivers a **secure, scalable, and realistic** enterprise network. By fulfilling all rubric requirements and adding advanced features like **NAT, BGP, and Port Security**, the design simulates a true production environment ready for deployment.



This project delivers a **secure, scalable, and realistic** enterprise network. By fulfilling all rubric requirements and adding advanced features like **NAT, BGP, and Port Security**, the design simulates a true production environment ready for deployment.
