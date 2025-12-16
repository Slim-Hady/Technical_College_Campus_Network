# COMPLETE TOPOLOGY & CONFIGURATION GUIDE

## Step 1: Hardware Setup in Packet Tracer
**CRITICAL:** Since we are using Serial (Red) cables, standard routers do not have these ports by default.
1.  **Turn OFF** the router physically (click the power switch).
2.  Drag and drop the **HWIC-2T** module into the empty slots.
3.  **Turn ON** the router.
4.  *Repeat this for ALL Routers.* (Note: The Main Router needs 2 or 3 HWIC-2T cards to have enough ports).

### Devices to Add:
* **Routers:** Add `Admin-Router` and `Library-Router`.
* **Switches:** Add Distribution & Access switches for Admin/Library.
* **PCs:** Populate Admin and Library buildings (80 hosts each).


##  Step 2: Physical Connections (Wiring Guide)
** IMPORTANT:** Check the exact port labels (e.g., `Se0/0/0` vs `Se0/1/0`) on your screen. The guide below assumes standard slot placement.

### 1. WAN Connections (Red Serial Cable)
**Use: Serial DCE/DTE Cable (Zig-zag Red)**

* **ISP Connection:**
    * `ISP-Router [Serial0/0/0]` ↔ `Main-Router [Serial0/0/0]`
* **CS Building Link:**
    * `Main-Router [Serial0/0/1]` ↔ `CS-Router [Serial0/0/0]`
* **Engineering Building Link:**
    * `Main-Router [Serial0/1/0]` ↔ `Eng-Router [Serial0/0/0]`
* **Admin Building Link:**
    * `Main-Router [Serial0/1/1]` ↔ `Admin-Router [Serial0/0/0]`
* **Library Building Link:**
    * `Main-Router [Serial0/2/0]`* (requires extra module) ↔ `Library-Router [Serial0/0/0]`

### 2. LAN Connections (Black Straight Cable ➖)
**Use: Copper Straight-Through**

* **Building Router to Switch:**
    * `Router [GigabitEthernet0/0]` ↔ `Distribution Switch [GigabitEthernet0/1]`
* **Between Switches:**
    * `Dist-Switch [Fa0/1]` ↔ `Access-Switch-1 [Gig0/1]`
    * `Dist-Switch [Fa0/2]` ↔ `Access-Switch-2 [Gig0/1]`
* **PCs:**
    * Connect PCs to Access Switches on FastEthernet ports.

---

## Step 3: Copy-Paste Configurations (CLI)

###  1. ISP Router (Simulated Internet)
```bash
enable
configure terminal
hostname ISP-Router
!
interface Serial0/0/0
 description Link-to-Main-Campus
 ip address 200.1.1.2 255.255.255.252
 no shutdown
 exit
!
interface Loopback0
 description Simulated-Google-Server
 ip address 8.8.8.8 255.255.255.255
 exit
!
router bgp 65001
 neighbor 200.1.1.1 remote-as 65000
 network 8.8.8.8 mask 255.255.255.255
 exit

```

###2. Main Campus Core Router (The Hub)**Note:** `clock rate 2000000` is added because this is the DCE side of the cable.

```bash
enable
configure terminal
hostname Main-Campus-Router

! --- WAN Interfaces (Serial) ---
interface Serial0/0/0
 description Link-to-ISP
 ip address 200.1.1.1 255.255.255.252
 clock rate 2000000
 ip nat outside
 no shutdown
 exit

interface Serial0/0/1
 description Link-to-CS
 ip address 10.0.10.1 255.255.255.252
 clock rate 2000000
 ip nat inside
 no shutdown
 exit

interface Serial0/1/0
 description Link-to-Engineering
 ip address 10.0.10.5 255.255.255.252
 clock rate 2000000
 ip nat inside
 no shutdown
 exit

interface Serial0/1/1
 description Link-to-Admin
 ip address 10.0.10.9 255.255.255.252
 clock rate 2000000
 ip nat inside
 no shutdown
 exit

interface Serial0/2/0
 description Link-to-Library
 ip address 10.0.10.13 255.255.255.252
 clock rate 2000000
 ip nat inside
 no shutdown
 exit

! --- Routing (OSPF + BGP) ---
router ospf 1
 router-id 1.1.1.1
 network 10.0.10.0 0.0.0.3 area 0
 network 10.0.10.4 0.0.0.3 area 0
 network 10.0.10.8 0.0.0.3 area 0
 network 10.0.10.12 0.0.0.3 area 0
 default-information originate
 exit

router bgp 65000
 neighbor 200.1.1.2 remote-as 65001
 network 10.0.0.0 mask 255.255.0.0
 exit

! --- NAT Configuration ---
access-list 1 permit 10.0.0.0 0.0.255.255
ip nat inside source list 1 interface Serial0/0/0 overload
ip route 0.0.0.0 0.0.0.0 200.1.1.2

```

### 3. CS Building Router```bash
enable
configure terminal
hostname CS-Router

interface Serial0/0/0
 description Link-to-Main
 ip address 10.0.10.2 255.255.255.252
 no shutdown
 exit

interface GigabitEthernet0/0
 description LAN-Gateway
 ip address 10.0.0.1 255.255.255.0
 no shutdown
 exit

! DHCP Server
ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp pool CS-POOL
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1
 dns-server 8.8.8.8
 exit

! OSPF
router ospf 1
 router-id 2.2.2.2
 network 10.0.10.0 0.0.0.3 area 0
 network 10.0.0.0 0.0.0.255 area 0
 exit
ip route 0.0.0.0 0.0.0.0 10.0.10.1

```

### 4. Engineering Building Router```bash
enable
configure terminal
hostname Eng-Router

interface Serial0/0/0
 description Link-to-Main
 ip address 10.0.10.6 255.255.255.252
 no shutdown
 exit

interface GigabitEthernet0/0
 description LAN-Gateway
 ip address 10.0.1.1 255.255.255.128
 no shutdown
 exit

! DHCP Server
ip dhcp excluded-address 10.0.1.1 10.0.1.10
ip dhcp pool ENG-POOL
 network 10.0.1.0 255.255.255.128
 default-router 10.0.1.1
 dns-server 8.8.8.8
 exit

! OSPF
router ospf 1
 router-id 3.3.3.3
 network 10.0.10.4 0.0.0.3 area 0
 network 10.0.1.0 0.0.0.127 area 0
 exit
ip route 0.0.0.0 0.0.0.0 10.0.10.5

```

###5. Admin Building Router```bash
enable
configure terminal
hostname Admin-Router

interface Serial0/0/0
 description Link-to-Main
 ip address 10.0.10.10 255.255.255.252
 no shutdown
 exit

interface GigabitEthernet0/0
 description LAN-Gateway
 ip address 10.0.1.129 255.255.255.128
 no shutdown
 exit

! DHCP Server
ip dhcp excluded-address 10.0.1.129 10.0.1.139
ip dhcp pool ADMIN-POOL
 network 10.0.1.128 255.255.255.128
 default-router 10.0.1.129
 dns-server 8.8.8.8
 exit

! OSPF
router ospf 1
 router-id 4.4.4.4
 network 10.0.10.8 0.0.0.3 area 0
 network 10.0.1.128 0.0.0.127 area 0
 exit
ip route 0.0.0.0 0.0.0.0 10.0.10.9

```

### 6. Library Building Router```bash
enable
configure terminal
hostname Library-Router

interface Serial0/0/0
 description Link-to-Main
 ip address 10.0.10.14 255.255.255.252
 no shutdown
 exit

interface GigabitEthernet0/0
 description LAN-Gateway
 ip address 10.0.2.1 255.255.255.128
 no shutdown
 exit

! DHCP Server
ip dhcp excluded-address 10.0.2.1 10.0.2.10
ip dhcp pool LIB-POOL
 network 10.0.2.0 255.255.255.128
 default-router 10.0.2.1
 dns-server 8.8.8.8
 exit

! OSPF
router ospf 1
 router-id 5.5.5.5
 network 10.0.10.12 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.127 area 0
 exit
ip route 0.0.0.0 0.0.0.0 10.0.10.13

```

---

##Step 4: Verification (Testing)1. **DHCP Check:** Go to any PC -> Desktop -> IP Configuration -> Click "DHCP". It should get an IP.
2. **Internal Ping:** From a CS PC, ping an Engineering PC (e.g., `ping 10.0.1.11`).
3. **Internet Ping:** From any PC, `ping 8.8.8.8`. If successful, NAT and BGP are working.

```

```
