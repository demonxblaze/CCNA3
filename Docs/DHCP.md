# DHCP Implementation in the Network

Documentation about the implementation of the DHCP service in the multi-site network.

## Overview

Dynamic Host Configuration Protocol (DHCP) was implemented across all three sites (Zagreb, Pula, and Split) to provide automatic IP address assignment to network devices. Each site uses a different approach:

- **Zagreb**: Dedicated DHCP Server (172.20.7.36) serving all VLANs
- **Pula**: Router RT1-Pl (172.20.11.105) providing DHCP services  
- **Split**: Router RT1-St (172.20.13.233) providing DHCP services

## Configuration Guidelines

The DHCP implementation follows these standards:

- Each VLAN has its own DHCP pool
- Default gateway points to the HSRP virtual IP address
- Domain name: isepacademy.ccna.itn.com (and isepacademy.isep.ipp.pt for Zagreb)
- DNS servers: 193.136.60.10 and 193.136.60.2 (ISEP Academy DNS)
- VoIP pools include option 150 (TFTP server) pointing to the local call manager
- VoIP pools have reduced lease time of 40 days for better management

## Zagreb Site Configuration

### DHCP Server (172.20.7.36)

**Excluded Addresses:**
```bash
ip dhcp excluded-address 172.20.7.105 172.20.7.106
ip dhcp excluded-address 172.20.7.109 172.20.7.110
ip dhcp excluded-address 172.20.7.1 172.20.7.6
ip dhcp excluded-address 172.20.7.65 172.20.7.68
ip dhcp excluded-address 172.20.7.33 172.20.7.36
ip dhcp excluded-address 172.20.4.1 172.20.4.4
ip dhcp excluded-address 172.20.5.1 172.20.5.4
ip dhcp excluded-address 172.20.0.1 172.20.0.3
ip dhcp excluded-address 172.20.2.1 172.20.2.4
ip dhcp excluded-address 172.20.6.1 172.20.6.4
```

**DHCP Pools:**

**Administration VLAN (VLAN 20):**
```bash
ip dhcp pool ADMINISTRATION
 network 172.20.4.0 255.255.255.0
 default-router 172.20.4.3
 domain-name isepacademy.isep.ipp.pt
 dns-server 193.136.60.10 193.136.60.2
```

**Staff VLAN (VLAN 30):**
```bash
ip dhcp pool STAFF
 network 172.20.5.0 255.255.255.0
 default-router 172.20.5.3
 domain-name isepacademy.isep.ipp.pt
 dns-server 193.136.60.10 193.136.60.2
```

**VoIP VLAN (VLAN 40):**
```bash
ip dhcp pool VOIP
 network 172.20.0.0 255.255.254.0
 default-router 172.20.0.3
 domain-name isepacademy.isep.ipp.pt
 dns-server 193.136.60.10 193.136.60.2
 option 150 ip 172.20.7.36
 lease 40
```

**Wireless VLAN (VLAN 50):**
```bash
ip dhcp pool WIRELESS
 network 172.20.2.0 255.255.254.0
 default-router 172.20.2.3
 domain-name isepacademy.isep.ipp.pt
 dns-server 193.136.60.10 193.136.60.2
```

**Guest VLAN (VLAN 60):**
```bash
ip dhcp pool GUEST
 network 172.20.6.0 255.255.255.0
 default-router 172.20.6.3
 domain-name isepacademy.isep.ipp.pt
 dns-server 193.136.60.10 193.136.60.2
```

## Pula Site Configuration (RT1-Pl)

**Excluded Addresses:**
```bash
ip dhcp excluded-address 172.20.11.97 172.20.11.98
ip dhcp excluded-address 172.20.11.105 172.20.11.106
ip dhcp excluded-address 172.20.10.129
ip dhcp excluded-address 172.20.9.1
ip dhcp excluded-address 172.20.11.1
ip dhcp excluded-address 172.20.8.1
ip dhcp excluded-address 172.20.10.1
ip dhcp excluded-address 172.20.11.65
```

**DHCP Pools:**

**Staff VLAN:**
```bash
ip dhcp pool STAFF
 network 172.20.10.128 255.255.255.128
 default-router 172.20.10.129
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
```

**Wireless VLAN:**
```bash
ip dhcp pool WIRELESS
 network 172.20.9.0 255.255.255.0
 default-router 172.20.9.1
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
```

**Guest VLAN:**
```bash
ip dhcp pool GUEST
 network 172.20.11.0 255.255.255.192
 default-router 172.20.11.1
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
```

**VoIP VLAN:**
```bash
ip dhcp pool VOIP
 network 172.20.10.0 255.255.255.128
 default-router 172.20.10.1
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
 option 150 ip 172.20.10.1
 lease 40
```

## Split Site Configuration (RT1-St)

**DHCP Pools:**

**Staff VLAN:**
```bash
ip dhcp pool STAFF
 network 172.20.12.0 255.255.255.128
 default-router 172.20.12.1
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
```

**Wireless VLAN:**
```bash
ip dhcp pool WIRELESS
 network 172.20.12.128 255.255.255.128
 default-router 172.20.12.129
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
```

**Guest VLAN:**
```bash
ip dhcp pool GUEST
 network 172.20.13.192 255.255.255.224
 default-router 172.20.13.193
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
```

**VoIP VLAN:**
```bash
ip dhcp pool VOIP
 network 172.20.13.64 255.255.255.192
 default-router 172.20.13.65
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
 option 150 ip 172.20.13.65
 lease 40
```

## DHCP Helper Configuration

DHCP relay (ip helper-address) is configured on all VLAN interfaces in the MLS switches to forward DHCP requests to the appropriate DHCP servers:

```bash
interface Vlan20
 ip helper-address 172.20.7.36
```

## Verification Commands

To verify DHCP functionality:

```bash
show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show ip dhcp statistics
```

## Key Features

- **Multi-site DHCP architecture** with site-specific servers
- **HSRP integration** with default gateways pointing to virtual IPs
- **VoIP support** with option 150 for TFTP server configuration
- **Excluded address ranges** to prevent conflicts with static assignments
- **Consistent DNS configuration** across all sites
