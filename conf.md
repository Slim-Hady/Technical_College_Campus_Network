# Topology : 
<img width="2538" height="1030" alt="image" src="https://github.com/user-attachments/assets/92034c39-cfc2-485f-a578-b160597da5d4" />


# conf : 

# ISB : 
<img width="805" height="896" alt="image" src="https://github.com/user-attachments/assets/6c555fda-fc76-4d21-b701-21fcb2978408" />

## Loopback : 
<img width="805" height="896" alt="image" src="https://github.com/user-attachments/assets/7eea9f4e-2b4e-4ea3-bca5-15203cafbf48" />


## bgp : 
<img width="805" height="896" alt="image" src="https://github.com/user-attachments/assets/445a70f9-bed0-4fd5-ab6c-7fbe8cf1cdcf" />


# Main Router : 

## Connect with Internet ( ISP ) : 
<img width="805" height="896" alt="image" src="https://github.com/user-attachments/assets/0b011b9c-b12d-42b1-8d60-986591703e06" />

## connect with CS : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/29f4ab16-8da6-493b-8c3a-0a337295db26" />

## connect with ENG : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/1f350276-e2ad-48b8-bca5-124a2715764b" />

## connect with Admin : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/c6b5b1c1-bf05-4c90-ac25-b10bb6549cfd" />

## connect with Lib : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/96a9e6fb-2e71-4ef9-9ee7-adfd81d78896" />

## OSPF : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/50da414d-4ac6-4df8-9d82-b8dcb20a8783" />

## bgp : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/d6d443cc-4ad9-46e8-a5da-a43eb58ed4c6" />

## ACL : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/3daf237c-8216-410d-a355-c23ebadd0f09" />


# CS Router : 
Interfaces : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/b7cf6533-4b32-4e8c-ae23-bfcd925abc21" />

Interfaces & DHCP : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/b9021a49-1452-48b4-8c0c-e37df3e2f187" />

OSPF :
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/43be9938-ecdf-4720-ba25-620dd0f71ec0" />

# ENG ROuter : 
interfaces : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/d5f2cab3-0e7e-45d4-83dd-3f440c26b00d" />

<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/e1cd5ded-d3f1-47c5-b8b2-08ac5fbc2fbd" />

# DHCP : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/6fa89936-d7ff-4d71-b6dc-8c80a4b4db8b" />

# OSPF : 
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/2a25b368-6789-44f0-8f96-6c2f42e7fbcd" />


# Admin Router :
## interfaces & OSPF: 
<img width="1003" height="890" alt="image" src="https://github.com/user-attachments/assets/24748828-f75f-49da-ad46-de5cc6a3ac7e" />


## DHCP :
<img width="810" height="895" alt="image" src="https://github.com/user-attachments/assets/c1f9c39c-0bc1-40ab-bfd6-cd261f32a80d" />


# Lib conf : 
<img width="1003" height="890" alt="image" src="https://github.com/user-attachments/assets/03673ce7-ce2a-43ca-ac0e-d1c4b2d607f7" />



# All conf : 
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

### CS-Router

```bash
enable
configure terminal
hostname CS-Router
interface Serial0/3/0
 ip address 10.0.10.2 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.0
 no shutdown
 exit
ip dhcp pool CS-POOL
 network 10.0.0.0 255.255.255.0
 default-router 10.0.0.1
 dns-server 8.8.8.8
 exit
router ospf 1
 router-id 2.2.2.2
 network 10.0.10.0 0.0.0.3 area 0
 network 10.0.0.0 0.0.0.255 area 0
 exit
end
write

```

### Eng-Router

```bash
enable
configure terminal
hostname Eng-Router
interface Serial0/3/1
 ip address 10.0.10.6 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/0
 ip address 10.0.1.1 255.255.255.128
 no shutdown
 exit
ip dhcp pool ENG-POOL
 network 10.0.1.0 255.255.255.128
 default-router 10.0.1.1
 dns-server 8.8.8.8
 exit
router ospf 1
 router-id 3.3.3.3
 network 10.0.10.4 0.0.0.3 area 0
 network 10.0.1.0 0.0.0.127 area 0
 exit
end
write

```

### Admin-Router

```bash
enable
configure terminal
hostname Admin-Router
default interface Serial0/1/0
interface Serial0/2/0
 ip address 10.0.10.10 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/0
 ip address 10.0.1.129 255.255.255.128
 no shutdown
 exit
ip dhcp pool ADMIN-POOL
 network 10.0.1.128 255.255.255.128
 default-router 10.0.1.129
 dns-server 8.8.8.8
 exit
router ospf 1
 router-id 4.4.4.4
 network 10.0.10.8 0.0.0.3 area 0
 network 10.0.1.128 0.0.0.127 area 0
 exit
end
write

```

### Library-Router

```bash
enable
configure terminal
hostname Library-Router
interface Serial0/1/0
 ip address 10.0.10.14 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/0
 ip address 10.0.2.1 255.255.255.128
 no shutdown
 exit
ip dhcp pool LIB-POOL
 network 10.0.2.0 255.255.255.128
 default-router 10.0.2.1
 dns-server 8.8.8.8
 exit
router ospf 1
 router-id 5.5.5.5
 network 10.0.10.12 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.127 area 0
 exit
end
write

```
