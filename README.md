# 2-Tier Enterprise Campus Architecture Network (NAT + DHCP + DNS + ACLs + L2 Security)

<img width="1920" height="902" alt="Topology" src="https://github.com/user-attachments/assets/84b13be9-0c30-46ce-a2f4-b9e8129b3784" />
<br>

Simplified 2-tier enterprise campus architecture featuring Access Layer, Distribution Layer, Edge/WAN Layer,NAT, DHCP, DNS, ACLs, L3 Routing, L2 switching, and L2 security via GNS3 VM. 
<br>

This lab was built using Cisco Virtual IOS L2 (vIOS L2) switch images, Cisco 7200 IOS (C7200-ADVENTERPRISEK9-M) router images, Virtual PCs, and VMware bridged networking (VMnet0)

## Limitations
1) This topology contains no traffic isolation, has single points of failure and lacks redundancy. Future improvements include VLAN separation, adding dual devices, FHRP, and link redundancy. 
<br>

2) A true public IP address for the simulated WAN segment wont be used but rather a private address because the lab environment is hosted behind my home router performing NAT via VMnet0 (The Cloud at top of topology), I will however still configure on NAT on the edge router.
   - In other words this topology demonstrates double NAT: <br>
            - Internal client traffic is translated by the Edge Router using PAT to 192.168.1.153 <br>
            - The traffic then exits through the VMware bridge/cloud <br>
            - The home router performs a second NAT translation to a real public IP before reaching the internet <br>

## This GNS3 project simulates a network with:

- NAT overload (PAT) for internet access
- ACLs to identify which traffic should be translated via PAT 
- DHCP services for internal subnets
- Layer 2 switching / Layer 3 routing 
- DHCP Snooping and Port Security (Sticky MAC)
---------------------------------------------------------------------------------------------------------------------------------

# Network Design

### WAN

#####  #IMPORTANT NOTE# <br>

The WAN-facing interface on the Edge Router does not use a true public IP address because the lab environment is connected behind my home router. The “public” WAN segment is simulated using the 192.168.1.0/24 private address space through VMware bridged networking (VMnet0).

- Cloud (Network Adapter VMnet0 - Bridged)
   - Home router performs a second NAT translation to a real public IP before reaching the internet
   - Ethernet 2 = Connected to Edge Router<br>

### Internal Networks

| Segment    | Network         | Gateway        |
| ----------------- | --------------- | -------------- |
| LAN 1             | 192.168.10.0/24 | 192.168.10.254 |
| LAN 2             | 192.168.20.0/24 | 192.168.20.254 |
| Core Transit | 10.0.0.0/30     | Point-to-point |
| Simulated WAN     | 192.168.1.0/24  | 192.168.1.1    |


---------------------------------------------------------------------------------------------------------------------------------
### NAT + ACLs

#### Implemented on Edge Router

- NAT Inside: g1/0 <br>
   - Facing LAN
- NAT Outside: g0/0 <br>
   - Facing WAN

NAT Overload (PAT) using named ACLs: <br>
ip access-list standard PAT <br>
permit 192.168.10.0 0.0.0.255 <br>
permit 192.168.20.0 0.0.0.255 <br>

- ip nat inside source list PAT interface GigabitEthernet0/0 overload
  - Allows multiple devices to share a single IP using different port numbers.
  
---------------------------------------------------------------------------------------------------------------------------------
### DHCP + DNS
#### Centralized on Distribution Router
- Excluded IP Addresses: 192.168.10.254 and 192.168.20.254
  - Reserved to keep static IP addresses on access switches.
- Pools:
   - POOL10: 192.168.10.0/24
      - DNS: 8.8.8.8
      - Domain: kelvin.lab.local
      - Default Gateway: 192.168.10.254
   - POOL20: 192.168.20.0/24
      - DNS: 8.8.8.8
      - Domain: kelvin.lab.local
      - Default Gateway: 192.168.20.254

### DHCP Verification: <BR>
The command to request an IP address from DHCP server is "ip dhcp" in GNS3 VM. In a Windows environment the command is "ipconfig /renew"

#### CLICK GIF
<img width="1000" height="506" alt="DHCP-Verification-subnet1" src="https://github.com/user-attachments/assets/9c13f5f7-fd66-4eeb-af05-4884a2def8a6" />

### DNS Verification:
<img width="894" height="309" alt="WAN-ISP-DNS Verification" src="https://github.com/user-attachments/assets/845a0a70-595b-4736-8327-528d8c4cfefc" />

---------------------------------------------------------------------------------------------------------------------------------
### Layer 2 Security
- DHCP Snooping enabled
  - Trusted ports configured toward router/uplinks
  - Limit rate of 4 DHCP packets per second on untrusted ports (end-users)
- Port Security (Sticky MAC)
   - Enabled on switch ports connected to end-users
   - Maximum MAC addresses allowed on each switch: 2
   - Violation mode set to restrict 
---------------------------------------------------------------------------------------------------------------------------------
### Features Implemented
- L3 Routing / L2 Switching
- DHCP services
- NAT overload (PAT)
- DNS reachability
- L2 security protections against spoofing and unauthorized rogue device plug-ins

---------------------------------------------------------------------------------------------------------------------------------

[Verifications](https://github.com/Kelvingonz/GNS3-Network-NAT-DHCP-DNS-ACLs-L2-Security-/tree/main/Verifications)
<br>
[Full Configurations](https://github.com/Kelvingonz/GNS3-Network-NAT-DHCP-DNS-ACLs-L2-Security-/tree/main/Configurations)

## Author

Kelvin Gonzalez
