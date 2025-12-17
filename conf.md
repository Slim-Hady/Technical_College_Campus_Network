### ISP-Router

```bash
enable
configure terminal
hostname ISP-Router
interface Serial0/0/0
ip address 200.1.1.2 255.255.255.252
clock rate 2000000
no shutdown
exit
interface Loopback0
ip address 8.8.8.8 255.255.255.255
exit
router bgp 65001
neighbor 200.1.1.1 remote-as 65000
network 8.8.8.8 mask 255.255.255.255
exit
end
write

```

### Main-Router

```bash
enable
configure terminal
hostname Main-Campus-Router
default interface Serial0/1/0
default interface Serial0/2/0
default interface Serial0/3/0
default interface Serial0/3/1
interface Serial0/0/0
ip address 200.1.1.1 255.255.255.252
ip nat outside
no shutdown
exit
interface Serial0/3/0
ip address 10.0.10.1 255.255.255.252
clock rate 2000000
ip nat inside
no shutdown
exit
interface Serial0/3/1
ip address 10.0.10.5 255.255.255.252
clock rate 2000000
ip nat inside
no shutdown
exit
interface Serial0/2/0
ip address 10.0.10.9 255.255.255.252
clock rate 2000000
ip nat inside
no shutdown
exit
interface Serial0/1/0
ip address 10.0.10.13 255.255.255.252
clock rate 2000000
ip nat inside
no shutdown
exit
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
access-list 1 permit 10.0.0.0 0.0.255.255
ip nat inside source list 1 interface Serial0/0/0 overload
ip route 0.0.0.0 0.0.0.0 200.1.1.2
end
write

```

# BUILDING 1: COMPUTER SCIENCE (CS)

### STEP 1: CS Distribution Switch
```
enable
configure terminal
hostname CS-Distribution-Switch

! Create VLANs for CS Department
vlan 10
 name CS-LABS
vlan 11
 name CS-FACULTY
vlan 12
 name CS-TAS
vlan 99
 name MGMT-CS
exit

! Management IP for remote access
interface vlan 99
 ip address 10.0.254.2 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.1

! Trunk to CS Router (Carries ALL VLANs)
interface GigabitEthernet1/0/1
 description Trunk-to-CS-Router
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,11,12,99
 no shutdown
 exit

! Trunks to Access Switches (Ports 2,3,4)
interface range GigabitEthernet1/0/2-4
 description Trunk-to-Access-Switches
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,11,12,99
 no shutdown
 exit

end
write memory
```
### TEST: 
```show vlan brief```

✅ Expected Output:
VLAN 10   CS-LABS         active
VLAN 11   CS-FACULTY      active  
VLAN 12   CS-TAS          active
VLAN 99   MGMT-CS         active
This shows all VLANs created successfully.

<img width="772" height="878" alt="image" src="https://github.com/user-attachments/assets/6b87566e-6516-4b01-9307-f4cb6207b2a7" />

### STEP 2: CS Access Switch 1 (LABS - VLAN 10)
```
enable
configure terminal
hostname CS-Access-Switch-1

! Create VLANs needed for this switch
vlan 10
 name CS-LABS
vlan 99
 name MGMT-CS
exit

! Trunk to Distribution Switch
interface GigabitEthernet0/1
 description Uplink-to-Distribution
 switchport mode trunk
 switchport trunk allowed vlan 10,99
 no shutdown
 exit

! Student Lab Ports (24 ports = 24 student PCs)
interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

! Management IP for SSH access
interface vlan 99
 ip address 10.0.254.11 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.1

! Enable SSH for secure management
ip domain-name cs.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
## TEST: 
```show port-security```

✅ Expected Output: 24 ports listed with "2" max addresses


<img width="772" height="878" alt="image" src="https://github.com/user-attachments/assets/0671e98d-8cea-4bde-bfba-63abb692896e" />


Verifies port security is active on all student ports.
### STEP 3: CS Access Switch 2 (FACULTY - VLAN 11)
```
enable
configure terminal
hostname CS-Access-Switch-2

vlan 11
 name CS-FACULTY
vlan 99
 name MGMT-CS
exit

interface GigabitEthernet0/1
 description Uplink-to-Distribution
 switchport mode trunk
 switchport trunk allowed vlan 11,99
 no shutdown
 exit

! Faculty Office Ports (Strict security - 1 device per port)
interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 11
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.12 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.1

ip domain-name cs.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
### TEST: 
```show interface trunk``


✅ Expected Output: 
Port        Mode         Encapsulation  Status        Native vlan
Gi0/1       on           802.1q         trunking      1

Confirms trunk is working properly.

<img width="772" height="878" alt="image" src="https://github.com/user-attachments/assets/ed621c36-8842-4365-be46-567b9ae26d3d" />

### STEP 4: CS Access Switch 3 (TEACHING ASSISTANTS - VLAN 12)

```
enable
configure terminal
hostname CS-Access-Switch-3

vlan 12
 name CS-TAS
vlan 99
 name MGMT-CS
exit

interface GigabitEthernet0/1
 description Uplink-to-Distribution
 switchport mode trunk
 switchport trunk allowed vlan 12,99
 no shutdown
 exit

! TA Office Ports
interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 12
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.13 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.1

ip domain-name cs.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
### TEST: 
```show ip interface brief```


✅ Expected Output: 
Interface    IP-Address      OK? Method Status
Vlan99       10.0.254.13     YES manual up

 Shows management interface is up with correct IP.

 <img width="772" height="878" alt="image" src="https://github.com/user-attachments/assets/ab2d52fe-e2f5-4e40-96e9-5c9bfae88c9b" />

# STEP 5: CS Router (GATEWAY FOR ALL CS VLANS)

```
enable
configure terminal
hostname CS-Router

!  CRITICAL: DHCP Exclusions (Prevent IP conflicts)
! Exclude gateway IPs and switch management IPs
ip dhcp excluded-address 10.0.0.1 10.0.0.10      ! Gateway + reserved
ip dhcp excluded-address 10.0.0.129 10.0.0.139   ! Faculty gateway + reserved
ip dhcp excluded-address 10.0.0.193 10.0.0.203   ! TAs gateway + reserved
ip dhcp excluded-address 10.0.254.1 10.0.254.20  ! All management IPs

! Remove old single-subnet DHCP pool
no ip dhcp pool CS-POOL

! Clear old configuration on G0/0
default interface GigabitEthernet0/0
interface GigabitEthernet0/0
 no ip address
 no shutdown
 exit

!  ROUTER-ON-A-STICK Configuration
! Single physical interface with multiple logical sub-interfaces
interface GigabitEthernet0/0.10
 description CS-LABS-VLAN
 encapsulation dot1Q 10
 ip address 10.0.0.1 255.255.255.128
 exit

interface GigabitEthernet0/0.11
 description CS-FACULTY-VLAN
 encapsulation dot1Q 11
 ip address 10.0.0.129 255.255.255.192
 exit

interface GigabitEthernet0/0.12
 description CS-TAS-VLAN
 encapsulation dot1Q 12
 ip address 10.0.0.193 255.255.255.192
 exit

interface GigabitEthernet0/0.99
 description MGMT-VLAN
 encapsulation dot1Q 99
 ip address 10.0.254.1 255.255.255.192
 exit

!  NEW DHCP Pools (One per VLAN)
ip dhcp pool CS-LABS
 network 10.0.0.0 255.255.255.128
 default-router 10.0.0.1
 dns-server 8.8.8.8
 exit

ip dhcp pool CS-FACULTY
 network 10.0.0.128 255.255.255.192
 default-router 10.0.0.129
 dns-server 8.8.8.8
 exit

ip dhcp pool CS-TAS
 network 10.0.0.192 255.255.255.192
 default-router 10.0.0.193
 dns-server 8.8.8.8
 exit

! WAN Connection to Main Router
interface Serial0/3/0
 description Link-to-Main-Campus
 ip address 10.0.10.2 255.255.255.252
 no shutdown
 exit

!  OSPF Routing Protocol
router ospf 1
 router-id 2.2.2.2
 ! Advertise WAN link
 network 10.0.10.0 0.0.0.3 area 0
 ! Advertise all CS VLANs
 network 10.0.0.0 0.0.0.127 area 0
 network 10.0.0.128 0.0.0.63 area 0
 network 10.0.0.192 0.0.0.63 area 0
 ! Advertise management network
 network 10.0.254.0 0.0.0.63 area 0
 exit

!  SSH Management
ip domain-name cs.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
## TESTS FOR CS ROUTER:

### Test 1:
```show ip interface brief```

✅ Expected: All sub-interfaces (G0/0.10, .11, .12, .99) show "up"

<img width="818" height="586" alt="image" src="https://github.com/user-attachments/assets/9f1e6b6b-f977-464d-a67a-5c3b59e69c3d" />


### Test 2: 
```show ip dhcp binding```

✅ Expected: Shows PCs that got IPs from DHCP pools

### Test 3: 
```show ip route```
 
✅ Expected: Should see OSPF routes to other buildings

### Test 4: 
```show ip ospf neighbor```

✅ Expected: Should see Main Router as neighbor (1.1.1.1)

<img width="872" height="701" alt="image" src="https://github.com/user-attachments/assets/bce51b2f-0679-46cd-a7f3-803e6ad7976c" />

# ACL : 
```
access-list 100 deny ip 10.0.0.0 0.0.0.127 10.0.0.128 0.0.0.63
access-list 100 deny ip 10.0.0.0 0.0.0.127 10.0.0.192 0.0.0.63
access-list 100 permit ip any any

interface GigabitEthernet0/0.10
 description Gateway-for-Labs
 ip access-group 100 in
 exit

end
write
```
### Testing : 

<img width="546" height="210" alt="image" src="https://github.com/user-attachments/assets/ee0c7590-b751-4601-8499-f1a8769649f6" />


# BUILDING 2: ENGINEERING (ENG) 

STEP 6: ENG Distribution Switch
```
enable
configure terminal
hostname ENG-Distribution-Switch

vlan 20
 name ENG-LABS
vlan 21
 name ENG-FACULTY
vlan 22
 name ENG-ADMIN
vlan 99
 name MGMT-ENG
exit

interface vlan 99
 ip address 10.0.254.66 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.65

interface GigabitEthernet1/0/1
 description Trunk-to-ENG-Router
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,21,22,99
 no shutdown
 exit

interface range GigabitEthernet1/0/2-4
 description Trunk-to-Access-Switches
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,21,22,99
 no shutdown
 exit

end
write memory
```
### TEST: 
```show vlan brief``` - Should see VLANs 20,21,22,99

<img width="947" height="311" alt="image" src="https://github.com/user-attachments/assets/938cbc77-22d9-4fe4-bbe9-f0b923a89ef6" />

### STEP 7: ENG Access Switch 1 (LABS - VLAN 20)

```
enable
configure terminal
hostname ENG-Access-Switch-1

vlan 20
 name ENG-LABS
vlan 99
 name MGMT-ENG
exit

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 20,99
 no shutdown
 exit

interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.70 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.65

ip domain-name eng.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
### STEP 8: ENG Access Switch 2 (FACULTY - VLAN 21)

```
enable
configure terminal
hostname ENG-Access-Switch-2

vlan 21
 name ENG-FACULTY
vlan 99
 name MGMT-ENG
exit

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 21,99
 no shutdown
 exit

interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 21
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.71 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.65

ip domain-name eng.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
### STEP 9: ENG Access Switch 3 (ADMIN - VLAN 22)

```
enable
configure terminal
hostname ENG-Access-Switch-3

vlan 22
 name ENG-ADMIN
vlan 99
 name MGMT-ENG
exit

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 22,99
 no shutdown
 exit

interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 22
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.72 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.65

ip domain-name eng.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
# STEP 10: ENG Router
```
enable
configure terminal
hostname Eng-Router

! DHCP Exclusions
ip dhcp excluded-address 10.0.1.1 10.0.1.10
ip dhcp excluded-address 10.0.1.65 10.0.1.75
ip dhcp excluded-address 10.0.1.97 10.0.1.105
ip dhcp excluded-address 10.0.254.65 10.0.254.75

! Remove old pool
no ip dhcp pool ENG-POOL

! Clear G0/0
default interface GigabitEthernet0/0
interface GigabitEthernet0/0
 no ip address
 no shutdown
 exit

! Sub-interfaces for VLANs
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.0.1.1 255.255.255.192
 exit

interface GigabitEthernet0/0.21
 encapsulation dot1Q 21
 ip address 10.0.1.65 255.255.255.224
 exit

interface GigabitEthernet0/0.22
 encapsulation dot1Q 22
 ip address 10.0.1.97 255.255.255.224
 exit

interface GigabitEthernet0/0.99
 encapsulation dot1Q 99
 ip address 10.0.254.65 255.255.255.192
 exit

! WAN Interface
interface Serial0/3/1
 ip address 10.0.10.6 255.255.255.252
 no shutdown
 exit

! DHCP Pools
ip dhcp pool ENG-LABS
 network 10.0.1.0 255.255.255.192
 default-router 10.0.1.1
 dns-server 8.8.8.8
 exit

ip dhcp pool ENG-FACULTY
 network 10.0.1.64 255.255.255.224
 default-router 10.0.1.65
 dns-server 8.8.8.8
 exit

ip dhcp pool ENG-ADMIN
 network 10.0.1.96 255.255.255.224
 default-router 10.0.1.97
 dns-server 8.8.8.8
 exit

! OSPF
router ospf 1
 router-id 3.3.3.3
 network 10.0.10.4 0.0.0.3 area 0
 network 10.0.1.0 0.0.0.63 area 0
 network 10.0.1.64 0.0.0.31 area 0
 network 10.0.1.96 0.0.0.31 area 0
 network 10.0.254.64 0.0.0.63 area 0
 exit

! SSH
ip domain-name eng.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
### TEST for ENG Router: 
```ping 10.0.10.5```

<img width="962" height="810" alt="image" src="https://github.com/user-attachments/assets/6febdefc-f633-4576-ac77-c38806aae4d9" />

<img width="962" height="810" alt="image" src="https://github.com/user-attachments/assets/19422d1c-24ff-4139-9450-61bb087b9e60" />

### ACL : 
```
enable
configure terminal

! Deny Students (10.0.1.0/26) -> Faculty (10.0.1.64/27)
access-list 100 deny ip 10.0.1.0 0.0.0.63 10.0.1.64 0.0.0.31
! Deny Students (10.0.1.0/26) -> Admin (10.0.1.96/27)
access-list 100 deny ip 10.0.1.0 0.0.0.63 10.0.1.96 0.0.0.31
! Permit Everything Else
access-list 100 permit ip any any

interface GigabitEthernet0/0.20
 description Gateway-for-ENG-Labs
 ip access-group 100 in
 exit

end
write
```
### testing : 
<img width="546" height="210" alt="image" src="https://github.com/user-attachments/assets/36c04494-88e3-4495-998a-b2ecd4093359" />

# bouns for now : 
Port Security (Layer 2 Protection)

Description: Implemented strict security policies at the access layer to prevent unauthorized devices (Rogue Laptops/Hubs) from connecting to the network.

  Configuration:
  Sticky MAC: Automatically learns connected device addresses.
  Violation Modes: Configured shutdown for Faculty (Zero Tolerance) and restrict for Students (Log & Drop).

  Impact: Prevents "Man-in-the-Middle" attacks and unauthorized network access.


<img width="705" height="211" alt="image" src="https://github.com/user-attachments/assets/a14ef108-c621-4bd1-b28d-6866ae743404" />

2. Management VLAN Isolation (VLAN 99)

    Description: Created a dedicated Management VLAN (VLAN 99) separate from user data traffic (Student/Faculty VLANs).

    Impact: Follows industry best practices for "Out-of-Band Management." It ensures that network administrators can still access and manage switches remotely via SSH even if the student network is congested or under attack.

<img width="735" height="129" alt="image" src="https://github.com/user-attachments/assets/a02009d4-8d0b-45ca-9e68-163614f9a58b" />

3. Spanning-Tree PortFast (Performance Optimization)

    Description: Enabled spanning-tree portfast on all end-user ports.

    Impact: Reduces the time for a port to become active from 30 seconds to sub-seconds. This eliminates DHCP timeouts when PCs are turned on, ensuring immediate connectivity
<img width="709" height="83" alt="image" src="https://github.com/user-attachments/assets/2b2baca9-d79c-43a9-b14c-b98feb690240" />

Since this interface is configured as an access port exclusively for VLAN 10, PortFast is correctly enabled only for that VLAN to optimize connectivity. VLAN 99 is listed as disabled because this port is isolated from the Management network, confirming that traffic segmentation is working as intended


# Admin : 
### Admin Dist : 
```
enable
configure terminal
hostname ADMIN-Distribution-Switch

vlan 30
 name ADMIN-STAFF
vlan 31
 name ADMIN-STUDENTS
vlan 32
 name ADMIN-ACCOUNTS
vlan 33
 name ADMIN-SERVICES
vlan 99
 name MGMT-ADMIN
exit

interface vlan 99
 ip address 10.0.254.130 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.129

interface GigabitEthernet1/0/1
 description Trunk-to-ADMIN-Router
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30-33,99
 no shutdown
 exit

interface range GigabitEthernet1/0/2-3
 description Trunk-to-Access-Switches
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30-33,99
 no shutdown
 exit

end
write memory
```

```show vlan brief```

<img width="791" height="405" alt="image" src="https://github.com/user-attachments/assets/ec273939-2303-432b-b4c7-8bbdad29d23e" />

```show interface trunk```

<img width="791" height="405" alt="image" src="https://github.com/user-attachments/assets/1cd29ca5-ee50-49ae-9cc1-808c6cb695e4" />

 ### ADMIN Access Switch 1 (STAFF + STUDENTS)

```
enable
configure terminal
hostname ADMIN-Access-Switch-1

vlan 30
 name ADMIN-STAFF
vlan 31
 name ADMIN-STUDENTS
vlan 99
 name MGMT-ADMIN
exit

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 30,31,99
 no shutdown
 exit

! First 12 ports for Staff (VLAN 30)
interface range FastEthernet0/1-12
 switchport mode access
 switchport access vlan 30
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

! Next 12 ports for Students (VLAN 31)
interface range FastEthernet0/13-24
 switchport mode access
 switchport access vlan 31
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.135 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.129

ip domain-name admin.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```

### TEST: 
```show vlan id 30```

<img width="774" height="212" alt="image" src="https://github.com/user-attachments/assets/abc3432c-3009-4fc4-8e82-d5a8042cbdd5" />

```show vlan id 31```
<img width="791" height="405" alt="image" src="https://github.com/user-attachments/assets/b2f586d4-bc6b-4372-a723-a7d778dffeed" />

``` show vlan ```
   Ports 1-12: VLAN 30 (Staff)

   Ports 13-24: VLAN 31 (Students)
    
<img width="791" height="405" alt="image" src="https://github.com/user-attachments/assets/fb2cda98-29ad-4e13-8093-e6323a983c15" />


### ADMIN Access Switch 2 (ACCOUNTS + SERVICES) :

```
enable
configure terminal
hostname ADMIN-Access-Switch-2

vlan 32
 name ADMIN-ACCOUNTS
vlan 33
 name ADMIN-SERVICES
vlan 99
 name MGMT-ADMIN
exit

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 32,33,99
 no shutdown
 exit

! Accounts Department Ports (VLAN 32)
interface range FastEthernet0/1-12
 switchport mode access
 switchport access vlan 32
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

! Services Ports (Printers, CCTV - VLAN 33)
interface range FastEthernet0/13-24
 switchport mode access
 switchport access vlan 33
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.136 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.129

ip domain-name admin.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
###  ADMIN Router : 
```
enable
configure terminal
hostname Admin-Router

ip dhcp excluded-address 10.0.1.129 10.0.1.140
ip dhcp excluded-address 10.0.1.193 10.0.1.203
ip dhcp excluded-address 10.0.1.225 10.0.1.232
ip dhcp excluded-address 10.0.1.241 10.0.1.248
ip dhcp excluded-address 10.0.254.129 10.0.254.140

default interface GigabitEthernet0/0
interface GigabitEthernet0/0
 no ip address
 no shutdown
 exit

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.1.129 255.255.255.192
 exit

interface GigabitEthernet0/0.31
 encapsulation dot1Q 31
 ip address 10.0.1.193 255.255.255.224
 exit

interface GigabitEthernet0/0.32
 encapsulation dot1Q 32
 ip address 10.0.1.225 255.255.255.240
 exit

interface GigabitEthernet0/0.33
 encapsulation dot1Q 33
 ip address 10.0.1.241 255.255.255.240
 exit

interface GigabitEthernet0/0.99
 encapsulation dot1Q 99
 ip address 10.0.254.129 255.255.255.192
 exit

interface Serial0/2/0
 ip address 10.0.10.10 255.255.255.252
 no shutdown
 exit

ip dhcp pool ADMIN-STAFF
 network 10.0.1.128 255.255.255.192
 default-router 10.0.1.129
 dns-server 8.8.8.8
 exit

ip dhcp pool ADMIN-STUDENTS
 network 10.0.1.192 255.255.255.224
 default-router 10.0.1.193
 dns-server 8.8.8.8
 exit

ip dhcp pool ADMIN-ACCOUNTS
 network 10.0.1.224 255.255.255.240
 default-router 10.0.1.225
 dns-server 8.8.8.8
 exit

ip dhcp pool ADMIN-SERVICES
 network 10.0.1.240 255.255.255.240
 default-router 10.0.1.241
 dns-server 8.8.8.8
 exit

router ospf 1
 router-id 4.4.4.4
 network 10.0.10.8 0.0.0.3 area 0
 network 10.0.1.128 0.0.0.63 area 0
 network 10.0.1.192 0.0.0.31 area 0
 network 10.0.1.224 0.0.0.15 area 0
 network 10.0.1.240 0.0.0.15 area 0
 network 10.0.254.128 0.0.0.63 area 0
 exit

ip domain-name admin.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
```show ip route connected```

<img width="791" height="405" alt="image" src="https://github.com/user-attachments/assets/db574dd5-f07f-4748-819f-16131d4d1624" />

Expected: Shows 5 connected networks (4 VLANs + MGMT)

```show ip route ```
<img width="791" height="405" alt="image" src="https://github.com/user-attachments/assets/f3fb3bcc-46f4-4cfd-8f04-0c2f48521adb" />

### Ping : 
<img width="791" height="405" alt="image" src="https://github.com/user-attachments/assets/6de1b33f-0529-487a-82a2-9bb9d50fb3b0" />

## DHCP : 
<img width="791" height="405" alt="image" src="https://github.com/user-attachments/assets/d71fad42-6f8e-44c0-826b-d0e5d176d97a" />

# LIBRARY : 
### LIB Distribution Switch :
```
enable
configure terminal
hostname LIB-Distribution-Switch

vlan 40
 name LIB-STUDENTS
vlan 41
 name LIB-STAFF
vlan 42
 name LIB-SERVERS
vlan 43
 name LIB-PRINTERS
vlan 99
 name MGMT-LIB
exit

interface vlan 99
 ip address 10.0.254.194 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.193

interface GigabitEthernet1/0/1
 description Trunk-to-LIB-Router
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 40-43,99
 no shutdown
 exit

interface range GigabitEthernet1/0/2-3
 description Trunk-to-Access-Switches
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 40-43,99
 no shutdown
 exit

end
write memory
```

### LIB Access Switch 1 (STUDENTS - VLAN 40) : 
```
enable
configure terminal
hostname LIB-Access-Switch-1

vlan 40
 name LIB-STUDENTS
vlan 99
 name MGMT-LIB
exit

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 40,99
 no shutdown
 exit

interface range FastEthernet0/1-24
 switchport mode access
 switchport access vlan 40
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.200 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.193

ip domain-name lib.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```

### LIB Access Switch 2 (STAFF + SERVERS + PRINTERS) : 

```
enable
configure terminal
hostname LIB-Access-Switch-2

vlan 41
 name LIB-STAFF
vlan 42
 name LIB-SERVERS
vlan 43
 name LIB-PRINTERS
vlan 99
 name MGMT-LIB
exit

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 41,42,43,99
 no shutdown
 exit

! Staff Ports (10 ports)
interface range FastEthernet0/1-10
 switchport mode access
 switchport access vlan 41
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

! Server Ports (5 ports)
interface range FastEthernet0/11-15
 switchport mode access
 switchport access vlan 42
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

! Printer Ports (5 ports)
interface range FastEthernet0/16-20
 switchport mode access
 switchport access vlan 43
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 spanning-tree portfast
 no shutdown
 exit

interface vlan 99
 ip address 10.0.254.201 255.255.255.192
 no shutdown
 exit
ip default-gateway 10.0.254.193

ip domain-name lib.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```
###  LIB Router : 
```
enable
configure terminal
hostname Library-Router

ip dhcp excluded-address 10.0.2.1 10.0.2.10
ip dhcp excluded-address 10.0.2.65 10.0.2.72
ip dhcp excluded-address 10.0.2.81 10.0.2.88
ip dhcp excluded-address 10.0.2.97 10.0.2.104
ip dhcp excluded-address 10.0.254.193 10.0.254.205

default interface GigabitEthernet0/0
interface GigabitEthernet0/0
 no ip address
 no shutdown
 exit

interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.0.2.1 255.255.255.192
 exit

interface GigabitEthernet0/0.41
 encapsulation dot1Q 41
 ip address 10.0.2.65 255.255.255.240
 exit

interface GigabitEthernet0/0.42
 encapsulation dot1Q 42
 ip address 10.0.2.81 255.255.255.240
 exit

interface GigabitEthernet0/0.43
 encapsulation dot1Q 43
 ip address 10.0.2.97 255.255.255.240
 exit

interface GigabitEthernet0/0.99
 encapsulation dot1Q 99
 ip address 10.0.254.193 255.255.255.192
 exit

interface Serial0/1/0
 ip address 10.0.10.14 255.255.255.252
 no shutdown
 exit

ip dhcp pool LIB-STUDENTS
 network 10.0.2.0 255.255.255.192
 default-router 10.0.2.1
 dns-server 8.8.8.8
 exit

ip dhcp pool LIB-STAFF
 network 10.0.2.64 255.255.255.240
 default-router 10.0.2.65
 dns-server 8.8.8.8
 exit

ip dhcp pool LIB-SERVERS
 network 10.0.2.80 255.255.255.240
 default-router 10.0.2.81
 dns-server 8.8.8.8
 exit

ip dhcp pool LIB-PRINTERS
 network 10.0.2.96 255.255.255.240
 default-router 10.0.2.97
 dns-server 8.8.8.8
 exit

router ospf 1
 router-id 5.5.5.5
 network 10.0.10.12 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.63 area 0
 network 10.0.2.64 0.0.0.15 area 0
 network 10.0.2.80 0.0.0.15 area 0
 network 10.0.2.96 0.0.0.15 area 0
 network 10.0.254.192 0.0.0.63 area 0
 exit

ip domain-name lib.college.local
crypto key generate rsa
username admin privilege 15 secret Admin@123
line vty 0 4
 transport input ssh
 login local
 exit

end
write memory
```


