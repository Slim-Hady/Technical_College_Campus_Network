# 📘 Project Requirements & IP Addressing Plan
**Project Name:** Technical College Campus Network
**Topology:** 4 Buildings (CS, Engineering, Administration, Library) + ISP Connection

---

## 1. Corrected VLSM Subnetting Table (IP Plan)
This table corrects the subnet mask errors found in the initial draft.

| Building | Hosts Needed | Network Address | Subnet Mask | CIDR | First Usable IP (Gateway) | Last Usable IP | Broadcast IP |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **CS Building** | 135 | `10.0.0.0` | `255.255.255.0` | **/24** | `10.0.0.1` | `10.0.0.254` | `10.0.0.255` |
| **Engineering** | 115 | `10.0.1.0` | `255.255.255.128` | **/25** | `10.0.1.1` | `10.0.1.126` | `10.0.1.127` |
| **Administration**| 80 | `10.0.1.128` | `255.255.255.128` | **/25** | `10.0.1.129` | `10.0.1.254` | `10.0.1.255` |
| **Library** | 80 | `10.0.2.0` | `255.255.255.128` | **/25** | `10.0.2.1` | `10.0.2.126` | `10.0.2.127` |

---

## 2.  WAN Links (Router-to-Router)
Point-to-Point connections using **Serial/Gigabit** links with **/30** masks.

### Internal Connections (Backbone)
| Connection | Network Address | Subnet Mask | Main Router IP | Building Router IP |
| :--- | :---: | :---: | :---: | :---: |
| **Main ↔ CS** | `10.0.10.0` | `255.255.255.252` | `10.0.10.1` | `10.0.10.2` |
| **Main ↔ Engineering** | `10.0.10.4` | `255.255.255.252` | `10.0.10.5` | `10.0.10.6` |
| **Main ↔ Admin** | `10.0.10.8` | `255.255.255.252` | `10.0.10.9` | `10.0.10.10` |
| **Main ↔ Library** | `10.0.10.12` | `255.255.255.252` | `10.0.10.13` | `10.0.10.14` |

### External Connection (Internet)
| Connection | Network Address | Subnet Mask | Main Router IP | ISP Router IP |
| :--- | :---: | :---: | :---: | :---: |
| **Main ↔ ISP** | `200.1.1.0` | `255.255.255.252` | `200.1.1.1` | `200.1.1.2` |

---

## 3. 🏢 Device Count & Requirements Per Building

###  CS Building (Computer Science)
* **Requirements:**
    * 4 Labs × 25 PCs = 100 PCs
    * 25 Doctor Offices = 25 PCs
    * 10 Teaching Assistants = 10 PCs
* **Total Hosts:** **135 Devices**
* **Switching Hardware:** 3 Access Switches + 1 Distribution Switch

### Engineering Building
* **Requirements:**
    * 3 Labs × 30 PCs = 90 PCs
    * 20 Doctor Offices = 20 PCs
    * 5 Administrators = 5 PCs
* **Total Hosts:** **115 Devices**
* **Switching Hardware:** 3 Access Switches + 1 Distribution Switch

### Administration Building
* **Requirements:**
    * Staff Offices = 40 PCs
    * Student Affairs = 20 PCs
    * Accounts = 10 PCs
    * CCTV & Printers = 10 Devices
* **Total Hosts:** **80 Devices**
* **Switching Hardware:** 2 Access Switches + 1 Distribution Switch

### 📚 Library Building
* **Requirements:**
    * Student PCs = 60 PCs
    * Staff PCs = 10 PCs
    * Servers = 5 Units
    * Printers = 5 Units
* **Total Hosts:** **80 Devices**
* **Switching Hardware:** 2 Access Switches + 1 Distribution Switch

---

## 4.  Total Network Equipment Summary

| Device Type | Quantity | Notes |
| :--- | :---: | :--- |
| **Core Router** | 1 | Main Campus Router (Cisco 2911 or ASR) |
| **Building Routers** | 4 | Connects each building to Core |
| **ISP Router** | 1 | Simulates Internet Connection |
| **Distribution Switches**| 4 | Layer 3 Switches (Cisco 3650) |
| **Access Switches** | 10 | Layer 2 Switches (Cisco 2960) |
| **End Devices** | 410 | PCs, Servers, Printers, etc. |
