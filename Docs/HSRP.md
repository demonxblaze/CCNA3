# Hot Standby Router Protocol (HSRP)

Documentation about the implementation of HSRP for gateway redundancy in the Zagreb site.

## Overview

Hot Standby Router Protocol (HSRP) is a Cisco proprietary redundancy protocol that provides gateway redundancy. In this network, HSRP is implemented on the Zagreb site's MLS switches (MLS1-Zg and MLS2-Zg) to provide high availability for all VLANs.

HSRP ensures that if one MLS switch fails, the other automatically takes over as the default gateway for the VLAN, providing seamless connectivity for end devices.

## Configuration

HSRP is configured on both MLS1-Zg and MLS2-Zg for all VLANs. MLS1-Zg is configured as the primary (higher priority) for all VLANs.

### MLS1-Zg Configuration (Primary)

**Access Point VLAN (VLAN 5):**
```bash
interface Vlan5
 description Access Point VLAN
 ip address 172.20.7.65 255.255.255.224
 standby 5 ip 172.20.7.67
 standby 5 priority 150
 standby 5 preempt
```

**Management VLAN (VLAN 6):**
```bash
interface Vlan6
 description Management VLAN
 ip address 172.20.7.3 255.255.255.224
 standby 6 ip 172.20.7.5
 standby 6 priority 150
 standby 6 preempt
```

**Servers VLAN (VLAN 10):**
```bash
interface Vlan10
 description Servers VLAN
 ip address 172.20.7.33 255.255.255.224
 standby 10 ip 172.20.7.35
 standby 10 priority 150
 standby 10 preempt
```

**Administration VLAN (VLAN 20):**
```bash
interface Vlan20
 description Admin Vlan
 ip address 172.20.4.1 255.255.255.0
 standby 20 ip 172.20.4.3
 standby 20 priority 150
 standby 20 preempt
```

**Staff VLAN (VLAN 30):**
```bash
interface Vlan30
 description Staff VLAN
 ip address 172.20.5.1 255.255.255.0
 standby 30 ip 172.20.5.3
 standby 30 priority 150
 standby 30 preempt
```

**VoIP VLAN (VLAN 40):**
```bash
interface Vlan40
 description VoIP VLAN
 ip address 172.20.0.1 255.255.254.0
 standby 40 ip 172.20.0.3
```

**Wireless VLAN (VLAN 50):**
```bash
interface Vlan50
 description Wireless VLAN
 ip address 172.20.2.1 255.255.254.0
 standby 50 ip 172.20.2.3
```

**Guest VLAN (VLAN 60):**
```bash
interface Vlan60
 description Guest VLAN
 ip address 172.20.6.1 255.255.255.0
 standby 60 ip 172.20.6.3
```

### MLS2-Zg Configuration (Backup)

MLS2-Zg is configured with the same HSRP groups but with lower priority (default 100), making it the backup device:

```bash
interface Vlan5
 standby 5 ip 172.20.7.67

interface Vlan6
 standby 6 ip 172.20.7.5

interface Vlan10
 standby 10 ip 172.20.7.35

interface Vlan20
 standby 20 ip 172.20.4.3

interface Vlan30
 standby 30 ip 172.20.5.3

interface Vlan40
 standby 40 ip 172.20.0.3

interface Vlan50
 standby 50 ip 172.20.2.3

interface Vlan60
 standby 60 ip 172.20.6.3
```

## HSRP Configuration Parameters

### Priority Configuration
- **MLS1-Zg Priority**: 150 (Primary)
- **MLS2-Zg Priority**: 100 (Default, Backup)

### Preemption
- **Preempt enabled** on MLS1-Zg to ensure it becomes active when available
- Allows automatic failback when the primary device recovers

### Virtual IP Addresses
Each VLAN has a dedicated virtual IP address that serves as the default gateway:

| VLAN | Description | Virtual IP | Physical IPs |
|------|-------------|------------|---------------|
| 5 | Access Points | 172.20.7.67 | MLS1: .65, MLS2: .66 |
| 6 | Management | 172.20.7.5 | MLS1: .3, MLS2: .4 |
| 10 | Servers | 172.20.7.35 | MLS1: .33, MLS2: .34 |
| 20 | Administration | 172.20.4.3 | MLS1: .1, MLS2: .2 |
| 30 | Staff | 172.20.5.3 | MLS1: .1, MLS2: .2 |
| 40 | VoIP | 172.20.0.3 | MLS1: .1, MLS2: .2 |
| 50 | Wireless | 172.20.2.3 | MLS1: .1, MLS2: .2 |
| 60 | Guest | 172.20.6.3 | MLS1: .1, MLS2: .2 |

## Integration with Other Services

### DHCP Integration
All DHCP pools are configured to use the HSRP virtual IP addresses as default gateways, ensuring clients always point to the active gateway.

### OSPF Integration
Both MLS switches participate in OSPF to advertise their networks and maintain routing table consistency.

## Verification Commands

**Check HSRP Status:**
```bash
show standby
show standby brief
show standby vlan [vlan-number]
```

**Check HSRP Configuration:**
```bash
show running-config interface vlan [vlan-number]
```

**Monitor HSRP State Changes:**
```bash
debug standby events
```

## Benefits

1. **High Availability**: Automatic failover in case of primary device failure
2. **Load Distribution**: Can be configured for load balancing across VLANs
3. **Seamless Failover**: End devices don't need reconfiguration
4. **Automatic Recovery**: Primary device automatically resumes active role when restored
5. **Network Redundancy**: Eliminates single point of failure for default gateway

## HSRP States

- **Active**: Primary device forwarding traffic
- **Standby**: Backup device ready to take over
- **Listen**: Device monitoring HSRP advertisements
- **Speak**: Device participating in HSRP election
- **Init**: Initial state when HSRP starts