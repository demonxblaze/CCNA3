# VoIP (Voice over IP)

Documentation about VoIP configuration in Cisco devices for the multi-site network.

## VoIP Overview

Voice over Internet Protocol (VoIP) is a technology that allows users to make voice calls using a broadband Internet connection instead of a regular (or analog) phone line. VoIP works by converting sound into digital voice communication and then transferring it through the Internet. 

In this solution we set up a VoIP network using Cisco IP Phones and used routers (Split & Pula) and Servers (Zagreb DHCP Server) as Call Managers.

## VoIP Architecture

### Multi-Site VoIP Design

**Call Manager Distribution:**
- **Zagreb Site**: DHCP Server (172.20.7.36) acts as Call Manager
- **Pula Site**: Router RT1-Pl (172.20.11.105) acts as Call Manager  
- **Split Site**: Router RT1-St (172.20.13.233) acts as Call Manager

### Phone Numbering Plan

**Dial Plan Format: X00N**
- **X**: Location Code
  - 1 = Zagreb
  - 2 = Pula
  - 4 = Split
- **00**: Fixed digits
- **N**: Extension Number (1-9)

**Example Phone Numbers:**
- Zagreb: 1001, 1002, 1003, etc.
- Pula: 2001, 2002, 2003, etc.
- Split: 4001, 4002, 4003, etc.

## VLAN Configuration for VoIP

### Voice VLAN Implementation

**VLAN 40 - VoIP VLAN:**
```bash
vlan 40
 name VOIP
```

**Voice VLAN Benefits:**
- **QoS Priority**: Voice traffic gets higher priority
- **Bandwidth Guarantee**: Dedicated bandwidth for voice
- **Security Isolation**: Voice traffic separated from data
- **Management**: Centralized voice network management

### Switchport Configuration

**Access Port with Voice VLAN:**
```bash
interface FastEthernet0/6
 description Access to VLAN 20 and Voice VLAN 40
 switchport access vlan 20        ! Data VLAN
 switchport voice vlan 40         ! Voice VLAN
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast
```

**Configuration Benefits:**
- **Dual VLAN**: Data and voice on same port
- **Automatic Detection**: CDP discovers IP phones
- **PortFast**: Immediate forwarding for phones
- **No Negotiation**: Prevents DTP attacks

## DHCP Configuration for VoIP

### VoIP DHCP Pools

**Zagreb VoIP Pool:**
```bash
ip dhcp pool VOIP
 network 172.20.0.0 255.255.254.0
 default-router 172.20.0.3
 domain-name isepacademy.isep.ipp.pt
 dns-server 193.136.60.10 193.136.60.2
 option 150 ip 172.20.7.36
 lease 40
```

**Pula VoIP Pool:**
```bash
ip dhcp pool VOIP
 network 172.20.10.0 255.255.255.128
 default-router 172.20.10.1
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
 option 150 ip 172.20.10.1
 lease 40
```

**Split VoIP Pool:**
```bash
ip dhcp pool VOIP
 network 172.20.13.64 255.255.255.192
 default-router 172.20.13.65
 domain-name isepacademy.ccna.itn.com
 dns-server 193.136.60.10 193.136.60.2
 option 150 ip 172.20.13.65
 lease 40
```

### DHCP Option 150

**TFTP Server Configuration:**
- **Purpose**: Provides firmware and configuration files to IP phones
- **Zagreb**: Points to DHCP Server (172.20.7.36)
- **Pula**: Points to Router (172.20.10.1)
- **Split**: Points to Router (172.20.13.65)

**Reduced Lease Time:**
- **40 days**: Shorter than standard DHCP leases
- **Benefits**: Better IP address management for mobile phones
- **Flexibility**: Easier moves and changes

## Quality of Service (QoS)

### Voice Traffic Prioritization

**Trust CoS on Voice Ports:**
```bash
interface FastEthernet0/6
 switchport voice vlan 40
 mls qos trust cos
```

**QoS Benefits:**
- **Low Latency**: Voice packets prioritized
- **Jitter Control**: Consistent packet timing
- **Bandwidth Guarantee**: Reserved bandwidth for voice
- **Packet Loss Prevention**: Critical voice packets protected

### Voice VLAN QoS Marking

**Automatic QoS Marking:**
- IP phones automatically mark voice traffic
- CoS value 5 for voice packets
- CoS value 3 for signaling packets
- DSCP EF (46) for voice payload

## Security Configuration

### VoIP Network Security

**VoIP VLAN ACL:**
```bash
ip access-list extended VOIP_NET
 remark VoIP network must not have access to the Internet
 permit ip any 10.0.0.0 0.255.255.255
 permit ip any 172.16.0.0 0.15.255.255 
 permit ip any 192.168.0.0 0.0.255.255
 deny   ip any any
```

**Security Policy:**
- **Internal Only**: VoIP traffic restricted to internal networks
- **No Internet Access**: Prevents unauthorized external calls
- **RFC 1918 Networks**: Allows communication between sites
- **Security Compliance**: Meets voice security requirements

### Voice Port Security

**Enhanced Port Security:**
```bash
interface FastEthernet0/6
 switchport port-security maximum 5
 switchport port-security aging time 5
 switchport port-security aging type inactivity
 switchport port-security
```

**Security Features:**
- **Maximum Devices**: Up to 5 MAC addresses (phone + PC)
- **Aging**: 5-minute inactivity timeout
- **Dynamic Learning**: Automatic MAC address learning
- **Violation Protection**: Shutdown on security violations

## Inter-Site VoIP Connectivity

### VoIP Over IPSec

**Call Flow:**
1. IP phone registers with local call manager
2. Inter-site calls routed through IPSec tunnels
3. Voice packets encrypted during transmission
4. Quality maintained across WAN links

### Bandwidth Considerations

**Voice Codec (G.711):**
- **Bandwidth**: 64 kbps per call
- **Packet Size**: 160 bytes payload
- **Overhead**: IP/UDP/RTP headers
- **Total**: ~87 kbps per call including overhead

**Tunnel Capacity:**
- Multiple simultaneous calls supported
- QoS prioritization across tunnels
- Bandwidth management for voice traffic

## Phone Configuration

### IP Phone Settings

**Network Configuration:**
- **VLAN**: Automatically assigned via CDP
- **IP Address**: DHCP assignment from voice pool
- **Default Gateway**: HSRP virtual IP address
- **DNS Servers**: ISEP Academy DNS servers
- **TFTP Server**: Option 150 from DHCP

**Call Manager Settings:**
- **Primary**: Local site call manager
- **Secondary**: Can be configured for redundancy
- **Registration**: Automatic via DHCP option 150

### Phone Features

**Supported Features:**
- **Directory Services**: Corporate directory access
- **Call Forwarding**: Advanced call routing
- **Voicemail**: Integrated voicemail system
- **Conference Calling**: Multi-party conferences
- **Call Transfer**: Blind and attended transfers

## Verification Commands

### VoIP VLAN Status
```bash
show vlan id 40
show interfaces switchport | include Voice
show cdp neighbors detail
```

### DHCP Pool Verification
```bash
show ip dhcp pool VOIP
show ip dhcp binding | include 172.20.0
show ip dhcp conflict
```

### QoS Verification
```bash
show mls qos interface [interface]
show mls qos interface [interface] statistics
```

### Phone Registration
```bash
show ephone registered
show ephone summary
show voice register pool
```

## Troubleshooting VoIP

### Common Issues

**Phone Not Getting IP Address:**
1. Check voice VLAN configuration
2. Verify DHCP pool configuration
3. Check PortFast and CDP
4. Verify trunk allows voice VLAN

**No Dial Tone:**
1. Check call manager connectivity
2. Verify option 150 configuration
3. Check phone registration status
4. Verify voice VLAN routing

**Poor Voice Quality:**
1. Check QoS configuration
2. Verify bandwidth availability
3. Check for network congestion
4. Verify codec selection

### Debug Commands
```bash
debug voice ccapi inout
debug ephone register
debug dhcp detail
debug cdp packets
```

## Best Practices Implemented

### Network Design
1. **Dedicated Voice VLAN**: Separation of voice and data traffic
2. **QoS Implementation**: Prioritization of voice packets
3. **Security Policies**: Restricted voice network access
4. **Redundancy**: Multiple call managers for reliability

### Configuration Standards
1. **Consistent Numbering**: Standard dial plan across sites
2. **DHCP Integration**: Automatic phone configuration
3. **Security Features**: Port security and access control
4. **Documentation**: Clear configuration documentation

### Performance Optimization
1. **PortFast Configuration**: Fast port transitions for phones
2. **CDP Optimization**: Reliable VLAN discovery
3. **Bandwidth Management**: Adequate capacity planning
4. **Monitoring**: Regular performance monitoring

## Benefits

1. **Cost Savings**: Reduced telecommunications costs
2. **Flexibility**: Easy moves, adds, and changes
3. **Features**: Advanced telephony features
4. **Integration**: Integration with data network
5. **Scalability**: Easy expansion across sites
6. **Management**: Centralized administration
7. **Security**: Secure voice communications
8. **Quality**: High-quality voice transmission

## Integration with Network Services

### HSRP Integration
- Default gateways point to HSRP virtual IPs
- Automatic failover for voice traffic
- Consistent routing across redundant paths

### OSPF Integration
- Voice subnets advertised via OSPF
- Inter-site voice routing
- Load balancing across multiple paths

### IPSec Integration
- Secure voice transmission between sites
- Encrypted signaling and media
- Quality preservation across tunnels
