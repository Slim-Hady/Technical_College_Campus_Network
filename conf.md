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
