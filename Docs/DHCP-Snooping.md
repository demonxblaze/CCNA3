# DHCP Snooping Security

Documentation about the DHCP snooping implementation for network security.

## Overview

DHCP Snooping is a security feature implemented across the network switches to prevent DHCP-based attacks such as rogue DHCP servers, DHCP starvation attacks, and man-in-the-middle attacks. It acts as a firewall between untrusted hosts and DHCP servers.

## DHCP Security Threats

### Common DHCP Attacks

**Rogue DHCP Server:**
- Unauthorized DHCP server providing malicious configuration
- Can redirect traffic through attacker's system
- May provide incorrect DNS servers or default gateways

**DHCP Starvation:**
- Exhausts DHCP pool by requesting all available addresses
- Causes denial of service for legitimate clients
- Often precedes rogue DHCP server attacks

**DHCP Spoofing:**
- Attacker impersonates legitimate DHCP server
- Provides malicious network configuration
- Can capture and redirect network traffic

## DHCP Snooping Implementation

### Global DHCP Snooping Configuration

**Relay Agent Verification:**
```bash
no ip dhcp snooping verify no-relay-agent-address
```

This command disables relay agent address verification, allowing DHCP packets from relay agents that don't add option 82 information.

### Trust Configuration

**Trusted Interfaces (Uplinks to DHCP Servers):**
```bash
interface GigabitEthernet0/1
 description Trunking Port to MLS
 ip dhcp snooping trust

interface GigabitEthernet1/0/2
 description Connection to SW1-Zg (EtherChannel member)
 ip dhcp snooping trust

interface GigabitEthernet1/0/3
 description Connection to SW1-Zg (EtherChannel member)
 ip dhcp snooping trust
```

**Trusted Interface Characteristics:**
- Allow DHCP server responses
- Allow DHCP relay agent messages
- No rate limiting applied
- Connected to legitimate DHCP infrastructure

### Rate Limiting Configuration

**Untrusted Access Ports:**
```bash
interface FastEthernet0/2
 description Access to VLAN 10
 ip dhcp snooping limit rate 5

interface FastEthernet0/6
 description Access to VLAN 20 and Voice VLAN 40
 ip dhcp snooping limit rate 5
```

**Rate Limiting Benefits:**
- **5 packets/second**: Prevents DHCP flooding
- **DoS Protection**: Limits impact of starvation attacks
- **Normal Operation**: Allows legitimate DHCP requests
- **Automatic Enforcement**: Drops excess packets

## Interface Trust States

### Trusted Interfaces

**Trust Criteria:**
- Connected to legitimate DHCP servers
- Connected to other network infrastructure devices
- Uplink ports and trunk connections
- DHCP relay agent connections

**Trusted Interface Configuration:**
```bash
interface [interface]
 ip dhcp snooping trust
```

### Untrusted Interfaces

**Untrusted Characteristics:**
- Access ports connected to end devices
- Default state for all interfaces
- Subject to rate limiting
- Cannot send DHCP server responses

**Default Behavior:**
- All interfaces untrusted by default
- Rate limiting automatically applied
- DHCP server messages blocked

## DHCP Snooping Database

### Binding Table

The switch maintains a binding table with entries for:
- **MAC Address**: Client hardware address
- **IP Address**: Assigned IP address
- **VLAN**: Client VLAN
- **Interface**: Connected switch port
- **Lease Time**: DHCP lease duration

### Binding Table Verification

**View Bindings:**
```bash
show ip dhcp snooping binding
show ip dhcp snooping binding interface [interface]
show ip dhcp snooping binding vlan [vlan-id]
```

### Database Storage

**Optional Persistent Storage:**
```bash
ip dhcp snooping database flash:dhcp_snooping.db
ip dhcp snooping database timeout 86400
```

## Integration with Other Security Features

### Dynamic ARP Inspection (DAI)

**ARP Inspection Trust:**
```bash
interface GigabitEthernet0/1
 ip arp inspection trust
 ip dhcp snooping trust
```

**Integration Benefits:**
- DHCP snooping provides IP-to-MAC bindings
- DAI uses bindings to validate ARP packets
- Prevents ARP spoofing attacks
- Maintains consistent security policies

### Port Security Integration

**Combined Security:**
```bash
interface FastEthernet0/6
 switchport port-security
 switchport port-security maximum 5
 ip dhcp snooping limit rate 5
```

**Layered Protection:**
- Port security controls MAC addresses
- DHCP snooping controls IP assignment
- Combined protection against various attacks

## VLAN-Specific Configuration

### DHCP Snooping Per VLAN

**Enable for Specific VLANs:**
```bash
ip dhcp snooping vlan 10,20,30,40,50,60
```

**VLAN Benefits:**
- Granular control per network segment
- Targeted protection for sensitive VLANs
- Resource optimization
- Flexible security policies

## Verification Commands

### DHCP Snooping Status
```bash
show ip dhcp snooping
show ip dhcp snooping statistics
show ip dhcp snooping track
```

### Binding Information
```bash
show ip dhcp snooping binding
show ip dhcp snooping binding summary
show ip dhcp snooping binding detail
```

### Interface Configuration
```bash
show ip dhcp snooping interface [interface]
show ip dhcp snooping trust
```

### Database Information
```bash
show ip dhcp snooping database
show ip dhcp snooping database detail
```

## Typical Output Examples

### DHCP Snooping Status
```
Switch# show ip dhcp snooping
Switch DHCP snooping is enabled
DHCP snooping is configured on following VLANs:
10,20,30,40,50,60
DHCP snooping is operational on following VLANs:
10,20,30,40,50,60
Insertion of option 82 is disabled
   circuit-id default format: vlan-mod-port
   remote-id: 0cd9.96e8.8800
Option 82 on untrusted port is not allowed
Verification of hwaddr field is enabled
Verification of giaddr field is enabled
DHCP snooping trust/rate is configured on the following Interfaces:

Interface                  Trusted    Allow option    Rate limit (pps)
-----------------------    -------    ------------    -----------------
GigabitEthernet0/1         yes        yes             unlimited
FastEthernet0/2            no         no              5
FastEthernet0/6            no         no              5
```

### Binding Table
```
Switch# show ip dhcp snooping binding
MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
------------------  ---------------  ----------  -------------  ----  --------------------
00:1B:44:11:3A:B7   172.20.4.100     86400       dhcp-snooping   20  FastEthernet0/6
00:50:56:BF:5A:B3   172.20.5.150     86400       dhcp-snooping   30  FastEthernet0/14
Total number of bindings: 2
```

## Troubleshooting DHCP Snooping

### Common Issues

**DHCP Clients Not Getting Addresses:**
1. Check trust configuration on uplink ports
2. Verify VLAN inclusion in snooping configuration
3. Check rate limiting on access ports
4. Verify DHCP server connectivity

**High CPU Usage:**
1. Check for DHCP flooding attacks
2. Verify rate limiting configuration
3. Monitor snooping statistics
4. Check binding table size

### Debug Commands
```bash
debug ip dhcp snooping packet
debug ip dhcp snooping event
debug ip dhcp snooping agent
```

### Error Messages
```bash
show logging | include DHCP_SNOOPING
show ip dhcp snooping statistics | include drop
```

## Best Practices Implemented

### Trust Configuration
1. **Minimal Trust**: Only necessary interfaces trusted
2. **Infrastructure Ports**: All uplinks and trunks trusted
3. **Documentation**: Clear trust policy documentation
4. **Regular Review**: Periodic trust configuration audit

### Rate Limiting
1. **Conservative Limits**: 5 pps for normal operation
2. **Consistent Application**: Same limits across similar ports
3. **Monitoring**: Regular statistics review
4. **Adjustment**: Rate limits adjusted based on usage patterns

### Integration Strategy
1. **Layered Security**: Combined with other security features
2. **Consistent Policies**: Aligned with overall security framework
3. **Performance**: Optimized for network performance
4. **Scalability**: Designed for network growth

## Configuration Templates

### Access Port Template
```bash
interface FastEthernet0/[port]
 description Access Port - VLAN [vlan]
 switchport access vlan [vlan]
 switchport mode access
 switchport port-security
 ip dhcp snooping limit rate 5
 spanning-tree portfast
```

### Trunk Port Template
```bash
interface GigabitEthernet0/[port]
 description Trunk to [device]
 switchport mode trunk
 switchport trunk native vlan 7
 ip dhcp snooping trust
 ip arp inspection trust
```

### Global DHCP Snooping Template
```bash
! Enable DHCP snooping globally
ip dhcp snooping
ip dhcp snooping vlan [vlan-range]

! Configure options
no ip dhcp snooping verify no-relay-agent-address
ip dhcp snooping database timeout 86400

! Optional: Configure database storage
ip dhcp snooping database flash:dhcp_snooping.db
```

## Security Benefits

### Attack Prevention
1. **Rogue DHCP Server**: Blocks unauthorized DHCP responses
2. **DHCP Starvation**: Rate limiting prevents pool exhaustion
3. **Man-in-the-Middle**: Validates DHCP server legitimacy
4. **IP Spoofing**: Maintains IP-to-MAC bindings

### Network Integrity
1. **Consistent Configuration**: Ensures proper network settings
2. **Centralized Control**: Manages IP assignment from trusted sources
3. **Audit Trail**: Provides binding history for security analysis
4. **Compliance**: Meets security policy requirements

### Operational Benefits
1. **Reduced Troubleshooting**: Prevents configuration conflicts
2. **Network Stability**: Eliminates rogue DHCP issues
3. **Security Monitoring**: Provides visibility into DHCP activity
4. **Automated Protection**: No manual intervention required

## Benefits Summary

1. **Security**: Protection against DHCP-based attacks
2. **Stability**: Prevents network configuration conflicts
3. **Visibility**: Provides IP assignment tracking
4. **Integration**: Works with other security features
5. **Performance**: Minimal impact on network performance
6. **Scalability**: Supports large network deployments
7. **Compliance**: Meets security best practices
8. **Automation**: Automatic threat detection and mitigation