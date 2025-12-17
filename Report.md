##  Project  Overview

<img width="2543" height="1053" alt="image" src="https://github.com/user-attachments/assets/15fbce59-e68b-4727-b01b-afa08ecd8c09" />


This project simulates a fully functional enterprise network connecting four major buildings (Computer Science, Engineering, Administration, Library) to the Internet. The design adopts a Hierarchical Network Model (Core, Distribution, Access) to ensure scalability, redundancy, and security.

Key Design Highlights:

    WAN Connectivity: Real-world ISP simulation using BGP and NAT.

    Campus Backbone: High-speed OSPF routing between buildings.

    Building LANs: VLAN segmentation with strict Port Security.

    Management: Centralized SSH management and automated IP addressing (DHCP).

(Place your Topology Image Here)
2. Phase 1: The Core Network (Internet & Gateway)

The "Brain" of the network that handles traffic between the Campus and the Outside World.
🌍 A. Internet Service Provider (ISP) Simulation

Since Packet Tracer is an isolated environment, we architected a simulation of the real internet.

    The "Internet" Target: Created a Loopback interface (8.8.8.8) to simulate a public DNS server (Google).

    BGP Routing: Configured Border Gateway Protocol (BGP) with AS 65001 to peer with the University. This simulates how real ISPs route traffic between autonomous systems.

🏢 B. Main Campus Router (The Edge Gateway)

This router acts as the border between the private university network and the public internet.

    Protocol Translation: Implements a hybrid routing strategy:

        BGP: Communicates externally with the ISP.

        OSPF: Communicates internally with the 4 Building Routers.

    NAT Overload (PAT):

        Problem: Internal IPs (10.x.x.x) are private and cannot route on the internet.

        Solution: configured NAT Overload. All thousands of student/faculty requests are translated to a single Public IP (200.1.1.1) before leaving the network.

3. Phase 2: Campus Connectivity (Routing Layer)

To connect the disparate buildings, we utilized OSPF (Open Shortest Path First) Area 0.

    Dynamic Routing: All Building Routers (CS, Eng, Admin, Lib) automatically "advertise" their subnets to the Main Router.

    Scalability: If a new lab is added to the "Library", OSPF automatically updates the routing table across the entire campus without manual intervention.

    Subnetting Strategy (VLSM):

        CS Building: 10.0.0.0/24 (Subnetted for Labs, Faculty, TAs).

        Engineering: 10.0.1.0/24 (Subnetted for Labs, Admin, Faculty).

        Admin & Library: 10.0.2.0/24 range.

4. Phase 3: Building Architecture (CS & Engineering)

Inside the buildings, we moved beyond simple connectivity to a Secure, Segmented, and Optimized design.
🔹 A. VLAN Segmentation & ROAS

Instead of a flat network, we separated traffic based on user roles using VLANs and Router-on-a-Stick:

    VLAN 10/20 (Labs): For Student PCs.

    VLAN 11/21 (Faculty): For Professors (Higher security).

    VLAN 99 (Management): Isolated traffic for Network Admins.

🔹 B. Switching Infrastructure

    Distribution Layer (Cisco 3650): Aggregates traffic from all floors/rooms and handles VLAN tagging via 802.1Q Trunks.

    Access Layer (Cisco 2960): Connects end-users. Configured with Rapid-PVST+ to ensure ports come online in sub-seconds (instant connectivity).

🔹 C. Security Hardening (The "Bonus" Features)

We treated the internal network as a "Zero Trust" environment:

    Port Security:

        Sticky MAC: Switches learn device MAC addresses automatically.

        Violation Modes: Faculty ports Shutdown if an unauthorized device connects. Student ports Restrict (block) intruders.

    ACLs (Access Control Lists):

        Configured on Building Routers to block Students from accessing the Faculty network, while still allowing them to access the Internet.

    SSH Encryption:

        Replaced insecure Telnet with SSH v2 for all device management.
