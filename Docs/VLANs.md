# VLAN Implementation

Documentation about the Virtual LAN (VLAN) implementation across the multi-site network.

## Overview

Virtual LANs (VLANs) are used to segment the network into logical groups, improving security, performance, and manageability. The network implements a standardized VLAN scheme across all sites with specific VLANs for different user groups and services.

## VLAN Design

### Zagreb Site VLANs

| VLAN ID | Name | Description | Subnet | Purpose |
|---------|------|-------------|---------|----------|
| 5 | APs | Access Points | 172.20.7.64/27 | Wireless Access Point Management |
| 6 | MANAGEMENT | Management | 172.20.7.0/27 | Network Device Management |
| 7 | NATIVE | Native VLAN | - | Trunk Native VLAN (Security) |
| 10 | SERVERS | Servers | 172.20.7.32/27 | Server Infrastructure |
| 20 | ADMINISTRATION | Administration | 172.20.4.0/24 | Administrative Staff |
| 30 | STAFF | Staff | 172.20.5.0/24 | Regular Staff Users |
| 40 | VOIP | Voice | 172.20.0.0/23 | IP Telephony |
| 50 | WIRELESS | Wireless | 172.20.2.0/23 | Wireless Clients |
| 60 | GUESTS | Guests | 172.20.6.0/24 | Guest Network Access |
| 99 | BLACKHOLE | Blackhole | - | Unused Ports Security |

### Pula Site VLANs

| VLAN ID | Name | Subnet | Purpose |
|---------|------|---------|----------|
| 30 | STAFF | 172.20.10.128/25 | Staff Users |
| 40 | VOIP | 172.20.10.0/25 | IP Telephony |
| 50 | WIRELESS | 172.20.9.0/24 | Wireless Clients |
| 60 | GUESTS | 172.20.11.0/26 | Guest Access |

### Split Site VLANs

| VLAN ID | Name | Subnet | Purpose |
|---------|------|---------|----------|
| 30 | STAFF | 172.20.12.0/25 | Staff Users |
| 40 | VOIP | 172.20.13.64/26 | IP Telephony |
| 50 | WIRELESS | 172.20.12.128/25 | Wireless Clients |
| 60 | GUESTS | 172.20.13.192/27 | Guest Access |

## VLAN Configuration

### Zagreb Switch Configuration (SW1-Zg)

**VLAN Creation:**
```bash
vlan 5
 name APs
!
vlan 6
 name MANAGEMENT
!
vlan 7
 name NATIVE
!
vlan 10
 name SERVERS
!
vlan 20
 name ADMINISTRATION
!
vlan 30
 name STAFF
!
vlan 40
 name VOIP
!
vlan 50
 name WIRELESS
!
vlan 60
 name GUESTS
!
vlan 99
 name BLACKHOLE
```

### Access Port Configuration

**Server Access (VLAN 10):**
```bash
interface FastEthernet0/1
 description Access to VLAN 10
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 switchport port-security aging time 2
 switchport port-security aging type inactivity
 spanning-tree portfast
 spanning-tree bpduguard enable
```

**Staff Access (VLAN 30):**
```bash
interface range FastEthernet0/5-12
 description Staff Ports
 switchport access vlan 30
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown
 spanning-tree portfast
 spanning-tree bpduguard enable
```

**VoIP Ports (VLAN 40):**
```bash
interface range FastEthernet0/13-20
 description VoIP Phones
 switchport access vlan 30
 switchport voice vlan 40
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 spanning-tree portfast
 spanning-tree bpduguard enable
```

### Trunk Configuration

**Inter-switch Trunks:**
```bash
interface GigabitEthernet1/0/1
 description Trunk to MLS1-Zg
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 7
 switchport trunk allowed vlan 5-7,10,20,30,40,50,60
 switchport mode trunk
 switchport nonegotiate
 channel-group 1 mode on
```

## Inter-VLAN Routing

Inter-VLAN routing is provided by the MLS switches in Zagreb and by the routers in Pula and Split sites.

### Zagreb Site
- **MLS1-Zg** and **MLS2-Zg** provide Layer 3 switching
- HSRP configured for gateway redundancy
- Each VLAN has a dedicated SVI (Switched Virtual Interface)

### Remote Sites
- **RT1-Pl** and **RT1-St** provide router-on-a-stick configuration
- Subinterfaces configured for each VLAN

## Security Features

### Port Security
- **Maximum MAC addresses**: 1 per access port, 2 per VoIP port
- **Violation action**: Shutdown
- **Aging**: 2 minutes inactivity-based
- **Sticky MAC**: Enabled for learned addresses

### VLAN Security
- **Native VLAN**: VLAN 7 for trunk security
- **Blackhole VLAN**: VLAN 99 for unused ports
- **VLAN Access Control**: ACLs applied per VLAN

### Spanning Tree Security
- **PortFast**: Enabled on access ports
- **BPDU Guard**: Enabled on access ports
- **Root Guard**: Configured on uplink ports

## Voice VLAN Configuration

VoIP phones are configured with:
- **Data VLAN**: User VLAN (typically VLAN 30)
- **Voice VLAN**: VLAN 40
- **Automatic detection**: CDP-based VLAN assignment
- **QoS marking**: Voice traffic prioritization

```bash
interface FastEthernet0/13
 switchport access vlan 30
 switchport voice vlan 40
 switchport mode access
 mls qos trust cos
```

## Verification Commands

**VLAN Information:**
```bash
show vlan brief
show vlan id [vlan-number]
show interfaces status
show interfaces trunk
```

**VLAN Connectivity:**
```bash
show ip interface brief
show ip route
ping [destination-ip]
```

**Security Status:**
```bash
show port-security
show port-security interface [interface]
show spanning-tree interface [interface] portfast
```

## Benefits

1. **Network Segmentation**: Logical separation of traffic
2. **Security**: Isolation between different user groups
3. **Performance**: Reduced broadcast domains
4. **Manageability**: Centralized VLAN administration
5. **Flexibility**: Easy moves, adds, and changes
6. **QoS**: Traffic prioritization per VLAN