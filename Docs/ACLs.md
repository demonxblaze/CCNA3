# Access Control Lists (ACLs)

Documentation about the implementation of Access Control Lists for network security and traffic control.

## Overview

Access Control Lists (ACLs) are implemented throughout the network to provide security, traffic filtering, and access control. The network uses both standard and extended ACLs for different purposes including SSH access control, SNMP security, VPN traffic classification, and VLAN-based security policies.

## ACL Types Implemented

### Standard ACLs
- **Purpose**: Source IP-based filtering
- **Usage**: SSH access, SNMP access, NAT matching
- **Location**: Close to destination

### Extended ACLs
- **Purpose**: Source/destination IP and protocol-based filtering
- **Usage**: VLAN security, VPN traffic classification
- **Location**: Close to source

## Standard ACL Configurations

### SSH Access Control (SSH-ACL)

Controls SSH access to network devices from specific management networks and tunnel interfaces.

```bash
ip access-list standard SSH-ACL
 permit 172.20.7.108 0.0.0.3    ! MLS management range
 permit 172.20.7.104 0.0.0.3    ! Router management range
 permit 10.0.0.0 0.0.0.3        ! Zagreb-Pula tunnel
 permit 10.0.0.4 0.0.0.3        ! Zagreb-Split tunnel
 permit 10.0.0.8 0.0.0.3        ! Pula-Split tunnel
 permit 172.20.11.104 0.0.0.7   ! Pula management
 permit 172.20.13.232 0.0.0.7   ! Split management
 permit 172.20.7.0 0.0.0.31     ! Zagreb management VLAN
 permit 172.20.10.128 0.0.0.127 ! Pula staff network
 permit 172.20.4.0 0.0.0.255    ! Zagreb admin network
 permit 172.20.5.0 0.0.0.255    ! Zagreb staff network
 permit 172.20.12.0 0.0.1.255   ! Split networks
 deny   any
```

**Applied to VTY lines:**
```bash
line vty 0 15
 access-class SSH-ACL in
 transport input ssh
```

### SNMP Access Control (SNMP-ACL)

Restricts SNMP access to only the LibreNMS monitoring server.

```bash
ip access-list standard SNMP-ACL
 permit 172.20.7.40             ! LibreNMS server
 deny   any
```

**Applied to SNMP service:**
```bash
snmp-server community cisco RO SNMP-ACL
```

### NAT Access Control (ACL 1)

Defines which internal networks are allowed to use NAT for internet access.

```bash
access-list 1 permit 172.20.0.0 0.0.7.255    ! All Zagreb VLANs
access-list 1 permit 172.20.7.108 0.0.0.3    ! MLS management
access-list 1 permit 172.20.7.104 0.0.0.3    ! Router management
access-list 1 deny   any
```

**Applied to NAT:**
```bash
ip nat inside source list 1 interface GigabitEthernet0/2 overload
```

## Extended ACL Configurations

### Guest Network Security (GUESTS_NET)

Restricts guest users to internet access only, blocking access to internal networks.

```bash
ip access-list extended GUESTS_NET
 remark GUEST networks must only have access to the Internet
 deny   ip any 10.0.0.0 0.255.255.255      ! Block RFC 1918 Class A
 deny   ip any 172.16.0.0 0.15.255.255     ! Block RFC 1918 Class B
 deny   ip any 192.168.0.0 0.0.255.255     ! Block RFC 1918 Class C
 permit ip any any                          ! Allow internet access
```

**Applied to Guest VLAN interface:**
```bash
interface Vlan60
 ip access-group GUESTS_NET in
```

### VoIP Network Security (VOIP_NET)

Restricts VoIP traffic to internal networks only, blocking internet access for security.

```bash
ip access-list extended VOIP_NET
 remark VoIP network must not have access to the Internet
 permit ip any 10.0.0.0 0.255.255.255      ! Allow RFC 1918 Class A
 permit ip any 172.16.0.0 0.15.255.255     ! Allow RFC 1918 Class B
 permit ip any 192.168.0.0 0.0.255.255     ! Allow RFC 1918 Class C
 deny   ip any any                          ! Block internet access
```

**Applied to VoIP VLAN interface:**
```bash
interface Vlan40
 ip access-group VOIP_NET in
```

### Crypto ACLs for IPSec

Define interesting traffic for IPSec tunnel establishment.

**Zagreb-Pula Tunnel:**
```bash
ip access-list extended CRYPTO-ACL-PULA
 permit gre host 172.16.201.80 host 172.16.201.77
 deny   ip any any
```

**Zagreb-Split Tunnel:**
```bash
ip access-list extended CRYPTO-ACL-SPLIT
 permit gre host 172.16.201.80 host 172.16.201.79
 deny   ip any any
```

**Applied to crypto maps:**
```bash
crypto map CROATIA 10 ipsec-isakmp
 match address CRYPTO-ACL-PULA
```

## Interface ACL Applications

### Router Interfaces

**Zagreb Router (RT1-Zg):**
```bash
interface GigabitEthernet0/0
 ip access-group 1 in           ! NAT ACL inbound
 ip access-group 101 out        ! Extended ACL outbound

interface GigabitEthernet0/1
 ip access-group 1 in           ! NAT ACL inbound
 ip access-group 101 out        ! Extended ACL outbound
```

### VLAN Interfaces (SVIs)

**Guest VLAN:**
```bash
interface Vlan60
 ip access-group GUESTS_NET in
```

**VoIP VLAN:**
```bash
interface Vlan40
 ip access-group VOIP_NET in
```

## ACL Security Policies

### Network Segmentation
1. **Guest Network Isolation**: Guests can only access internet
2. **VoIP Security**: Voice traffic restricted to internal networks
3. **Management Access**: Limited SSH access from specific networks
4. **Monitoring Security**: SNMP restricted to monitoring server

### Traffic Flow Control
1. **Inbound Filtering**: Controls traffic entering interfaces
2. **Outbound Filtering**: Controls traffic leaving interfaces
3. **Bidirectional Security**: Applied on VLAN interfaces

### Administrative Access
1. **SSH-Only Access**: Telnet blocked, only SSH allowed
2. **Source-Based Control**: Access limited to specific source networks
3. **Management VLAN Security**: Dedicated ACLs for management traffic

## Verification Commands

**ACL Configuration:**
```bash
show access-lists
show access-lists [acl-name]
show ip access-lists
```

**ACL Application:**
```bash
show ip interface [interface] | include access list
show interfaces [interface] | include access list
```

**ACL Statistics:**
```bash
show access-lists [acl-name] | include matches
```

**Clear ACL Counters:**
```bash
clear access-list counters [acl-name]
```

## Best Practices Implemented

### ACL Design Principles
1. **Explicit Deny**: All ACLs end with implicit deny any
2. **Most Specific First**: More specific rules before general rules
3. **Logging**: Important deny statements include logging
4. **Documentation**: Remarks explain ACL purpose

### Security Hardening
1. **Principle of Least Privilege**: Only necessary access granted
2. **Defense in Depth**: Multiple layers of ACL filtering
3. **Regular Review**: ACLs reviewed for effectiveness
4. **Standard Templates**: Consistent ACL patterns across devices

### Performance Considerations
1. **Efficient Ordering**: Most-hit rules first
2. **Standard vs Extended**: Right ACL type for the job
3. **Interface Placement**: ACLs applied closest to traffic source
4. **Wildcard Masks**: Efficient subnet matching

## Troubleshooting

### Common Issues
1. **Implicit Deny**: Traffic blocked by default deny
2. **Order Dependency**: Rules processed sequentially
3. **Wildcard Masks**: Incorrect mask calculations
4. **Interface Direction**: Wrong inbound/outbound application

### Debugging Commands
```bash
debug ip packet [acl-number] [detail]
show logging | include %SEC
```

### Testing Methodology
1. **Before/After Testing**: Verify connectivity before ACL changes
2. **Incremental Implementation**: Add rules gradually
3. **Traffic Analysis**: Use packet captures to verify filtering
4. **User Acceptance**: Validate with end users

## Benefits

1. **Network Security**: Prevents unauthorized access
2. **Traffic Control**: Manages network traffic flow
3. **Compliance**: Meets security policy requirements
4. **Performance**: Optimizes network resource usage
5. **Segmentation**: Isolates different user groups
6. **Incident Response**: Provides traffic blocking capability