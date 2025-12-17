## 1. Project Topology Overview

We designed a hierarchical network for a University Campus consisting of four separate buildings:

1. **CS (Computer Science)**
2. **Engineering**
3. **Administration**
4. **Library**

These buildings connect to a **Main Campus Router**, which connects to an **ISP Router** to simulate Internet access. The internal network uses a multi-layer switching architecture (Distribution and Access layers).

---

## 2. Phase 1: The Internet Service Provider (ISP)

### Architecture

We started by placing a router named **ISP**. Since Packet Tracer does not have real internet access, we had to simulate it.

### Configuration Details

1. **Loopback Interface (The Internet Simulation):**
We created a virtual interface called `Loopback0` and assigned it the IP `8.8.8.8`. This simulates a public server (like Google DNS). This gives us a target to test our connectivity against.
* **Command:**
```
interface Loopback0
ip address 8.8.8.8 255.255.255.255

```




2. **Connection to Campus:**
We used a Serial connection with a Public IP (`200.1.1.2/30`) to connect to the University.
* **Command:**
```
interface Serial0/0/0
ip address 200.1.1.2 255.255.255.252
clock rate 2000000
no shutdown

```




3. **BGP Routing (External Protocol):**
We configured **BGP (Border Gateway Protocol)** with Autonomous System (AS) number **65001**. We peered it with the University's AS (**65000**).
* **Why:** BGP is the standard protocol for routing between different organizations on the Internet.
* **Command:**
```
router bgp 65001
neighbor 200.1.1.1 remote-as 65000
network 8.8.8.8 mask 255.255.255.255

```





---

## 3. Phase 2: The Main Campus Router (The Gateway)

### Architecture

This router is the **Edge Gateway**. It is the most critical device because it handles the traffic between the internal buildings and the outside world.

### Configuration Details

1. **WAN Interface (Facing ISP):**
Assigned Public IP `200.1.1.1` and marked it as the NAT Outside interface.
* **Command:**
```
interface Serial0/0/0
ip address 200.1.1.1 255.255.255.252
ip nat outside

```




2. **LAN Interfaces (Facing Buildings):**
Connected to the 4 building routers using Serial cables. Each interface was assigned a Private IP (e.g., `10.0.10.1`) and marked as NAT Inside.
* **Command:**
```
interface Serial0/3/0
ip address 10.0.10.1 255.255.255.252
ip nat inside

```




3. **NAT Overload (Port Address Translation):**
* **Why:** Internal University IPs (`10.x.x.x`) are **Private**. They cannot route on the Internet. We used NAT to translate all internal private IPs to the single Public IP (`200.1.1.1`).
* **Command:**
```
access-list 1 permit 10.0.0.0 0.255.255.255
ip nat inside source list 1 interface Serial0/0/0 overload

```




4. **Hybrid Routing (BGP + OSPF):**
* **BGP:** To talk to the ISP.
```
router bgp 65000
neighbor 200.1.1.2 remote-as 65001

```


* **OSPF:** To talk to the internal buildings.
```
router ospf 1
network 10.0.10.0 0.0.0.255 area 0
default-information originate

```


* **Why:** We separate internal routing (OSPF is fast) from external routing (BGP is secure and scalable).



---

## 4. Phase 3: The Building Routers (Internal Network)

### Architecture

We deployed 4 routers, one for each building. Each router acts as the gateway for its specific department.

### Configuration Details (Applied to CS, Eng, Admin, Lib)

1. **Subnetting:**
We assigned a unique IP range to each building to avoid conflicts.
* CS: `10.0.0.0/24`
* Eng: `10.0.1.0/25`
* Admin: `10.0.1.128/25`
* Lib: `10.0.2.0/25`


2. **OSPF Configuration:**
We enabled OSPF Area 0 on all building routers.
* **Why:** This allows the Main Router to automatically learn about the networks inside each building without writing manual static routes.
* **Command:**
```
router ospf 1
network 10.0.0.0 0.0.0.255 area 0

```




3. **DHCP Server (Automation):**
We configured the routers to automatically assign IP addresses to PCs.
* **Why:** To support scalability. We do not need to manually configure PCs.
* **Command:**
```
ip dhcp pool BUILDING_POOL
network 10.0.0.0 255.255.255.0
default-router 10.0.0.1
dns-server 8.8.8.8

```





---

## 5. Phase 4: LAN Infrastructure (Switching Layer)

### Architecture

Inside each building, we implemented a 2-Tier Switching Architecture.

1. **Distribution Layer (Cisco 3650 Multilayer Switch):**
* **Quantity:** 1 per building (4 Total).
* **Role:** It connects the Router to the Access Switches. It handles high-speed traffic aggregation.
* **Why:** To reduce the load on the router interfaces and provide a central point for the building's network.


2. **Access Layer (Cisco 2960 Switch):**
* **Quantity:** 3 per building (12 Total).
* **Role:** These switches connect directly to the end-user devices (PCs).
* **Layout:** We simulated **3 distinct Labs/Rooms** in each building. Each room has its own switch.


3. **End Devices:**
* **Quantity:** We placed 6 PCs per building for simulation purposes (Total 24 PCs).
* **Note:** Due to DHCP, the network supports up to 253 devices per building immediately without config changes.



---

## 6. Phase 5: Verification and Testing

To ensure the network is fully functional, we performed the following tests:

### Test 1: DHCP Verification

* **Action:** We set the PCs to "DHCP" mode.
* **Result:** All PCs successfully received an IP address (e.g., `10.0.0.x`), a Subnet Mask, a Gateway, and a DNS Server.
* **Meaning:** The Switch connection and the Router's DHCP pool are working.

### Test 2: Internal Connectivity (Intranet)

* **Action:** We performed a Ping from a PC in the **Library** to a PC in **Engineering**.
* **Result:** `Reply from 10.0.1.x: bytes=32 time=1ms`.
* **Meaning:** OSPF is correctly routing traffic between buildings through the Main Router.

### Test 3: Internet Connectivity (NAT & BGP)

* **Action:** We performed a Ping from a PC to `8.8.8.8` (The ISP Loopback).
* **Result:** `Reply from 8.8.8.8: bytes=32 time=...`
* **Meaning:**
1. The packet left the PC.
2. Routed to Main Router via OSPF.
3. Translated by NAT (Private -> Public).
4. Sent to ISP via BGP.
5. Returned successfully.



### Test 4: Routing Table Verification

* **Action:** We used the command `show ip route` on the Main Router.
* **Result:** We observed `O` (OSPF) routes for all internal buildings and `B` (BGP) connections to the ISP.

---

## 7. Project Summary (Bonus Features)

This project is an **Enterprise-Grade** simulation. It goes beyond basic connectivity by implementing:

1. **NAT Overload:** To allow real-world internet simulation using Private IPs.
2. **Hybrid Routing (OSPF + BGP):** Separating internal campus routing from external internet routing.
3. **On-Router DHCP:** Fully automating user management.
4. **Hierarchical Switching:** Using Distribution and Access layers for realistic building design.
