# Rapid Spanning Tree Protocol (RSTP)

Documentation about the RSTP implementation for loop prevention and network redundancy.

## Overview

Rapid Spanning Tree Protocol (RSTP) is implemented across the network to prevent Layer 2 loops while providing network redundancy. RSTP is an evolution of the original Spanning Tree Protocol (STP) that provides faster convergence and improved performance in modern switched networks.

## RSTP Implementation

### Protocol Selection

**RSTP Configuration:**
```bash
spanning-tree mode rapid-pvst
```

**Rapid Per-VLAN Spanning Tree Plus (Rapid PVST+) Features:**
- Separate spanning tree instance per VLAN
- Rapid convergence (typically under 1 second)
- Backward compatibility with legacy STP
- VLAN load balancing capabilities

### Global RSTP Configuration

**Extended System ID:**
```bash
spanning-tree extend system-id
```

This enables extended system ID which:
- Incorporates VLAN ID into bridge priority
- Allows unique bridge ID per VLAN
- Supports more than 64 VLAN instances

## Port Fast Configuration

### Default Port Fast
```bash
spanning-tree portfast default
```

**Port Fast Benefits:**
- Immediately transitions access ports to forwarding state
- Eliminates 30-second delay for end devices
- Improves user experience during device connection
- Reduces unnecessary STP recalculation

### Per-Interface Port Fast
```bash
interface FastEthernet0/6
 spanning-tree portfast
```

Applied to all access ports connecting end devices:
- Workstations
- IP phones
- Servers
- Printers

## BPDU Guard Configuration

### Global BPDU Guard
```bash
spanning-tree portfast bpduguard default
```

**BPDU Guard Protection:**
- Automatically enabled on all PortFast ports
- Shuts down ports receiving BPDUs
- Prevents accidental switch connections
- Protects against bridging loops

### BPDU Guard Operation
```
Normal Operation: Access Port → End Device (No BPDUs)
Security Violation: Access Port → Switch (BPDU Received) → Port Shutdown
```

## Network Topology and Redundancy

### Zagreb Site Topology

**Primary Links:**
- MLS1-Zg ↔ MLS2-Zg (EtherChannel)
- SW1-Zg ↔ MLS1-Zg (Trunk)
- SW2-Zg ↔ MLS2-Zg (Trunk)

**Redundant Links:**
- Cross-connections between switches for redundancy
- EtherChannel provides load balancing and redundancy

### Root Bridge Selection

**Bridge Priority Configuration:**
```bash
spanning-tree vlan 1-99 priority 4096    ! MLS1-Zg (Primary Root)
spanning-tree vlan 1-99 priority 8192    ! MLS2-Zg (Secondary Root)
```

**Root Bridge Hierarchy:**
1. **MLS1-Zg**: Primary root for all VLANs
2. **MLS2-Zg**: Secondary root for all VLANs
3. **Access Switches**: Higher priority values

## RSTP Port Roles

### Port Role Assignments

**Root Port:**
- Port with lowest cost path to root bridge
- One per non-root switch
- Automatically selected by RSTP

**Designated Port:**
- Port designated to forward traffic for a segment
- All ports on root bridge are designated
- One designated port per network segment

**Alternate Port:**
- Backup path to root bridge
- Blocked in normal operation
- Rapidly becomes root port if needed

**Backup Port:**
- Backup for designated port
- Less common in modern networks
- Provides redundancy for same segment

### Port States

**Learning → Forwarding:**
- RSTP eliminates blocking and listening states
- Ports transition faster between states
- Edge ports (PortFast) immediately forward

## Security Features

### PortFast Security
```bash
interface FastEthernet0/6
 switchport access vlan 30
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
```

**Security Benefits:**
- **PortFast**: Fast transition for end devices
- **BPDU Guard**: Protection against switch loops
- **Access Mode**: Prevents VLAN hopping
- **Port Security**: MAC address learning control

### Unused Port Security
```bash
interface FastEthernet0/21
 description Unused Interface
 switchport access vlan 99
 switchport mode access
 shutdown
```

**Unused Port Protection:**
- Assigned to blackhole VLAN (99)
- Administratively shutdown
- Prevents unauthorized access

## EtherChannel Integration

### LACP with RSTP
```bash
interface GigabitEthernet1/0/2
 channel-group 1 mode on
 spanning-tree portfast trunk
```

**EtherChannel Benefits with RSTP:**
- Single logical link reduces STP complexity
- Load balancing across physical links
- Faster convergence with fewer topology changes
- Increased bandwidth without STP blocking

## VLAN-Specific RSTP

### Per-VLAN Root Selection

**Load Balancing Configuration:**
```bash
! MLS1-Zg - Primary root for VLANs 1-50
spanning-tree vlan 1-50 priority 4096

! MLS2-Zg - Primary root for VLANs 51-99
spanning-tree vlan 51-99 priority 4096
spanning-tree vlan 1-50 priority 8192
```

This provides:
- Load distribution across uplinks
- Redundancy for all VLANs
- Optimal path utilization

## Verification Commands

### RSTP Status
```bash
show spanning-tree
show spanning-tree summary
show spanning-tree brief
```

### Per-VLAN Information
```bash
show spanning-tree vlan [vlan-id]
show spanning-tree vlan [vlan-id] detail
```

### Port-Specific Information
```bash
show spanning-tree interface [interface]
show spanning-tree interface [interface] detail
```

### Root Bridge Information
```bash
show spanning-tree root
show spanning-tree bridge
```

### PortFast and BPDU Guard Status
```bash
show spanning-tree interface [interface] portfast
show spanning-tree summary totals
```

## Troubleshooting RSTP

### Common Issues

**STP Loops:**
```bash
show spanning-tree inconsistentports
show spanning-tree interface [interface] detail
```

**PortFast Issues:**
```bash
show spanning-tree interface [interface] portfast
show errdisable recovery
```

**Root Bridge Problems:**
```bash
show spanning-tree root
show spanning-tree bridge priority
```

### Debug Commands
```bash
debug spanning-tree events
debug spanning-tree bpdu
debug spanning-tree portfast
```

## RSTP Convergence

### Convergence Times

**Traditional STP:**
- Blocking → Listening: 20 seconds
- Listening → Learning: 15 seconds
- Learning → Forwarding: 15 seconds
- **Total**: 50 seconds

**RSTP Convergence:**
- Point-to-point links: < 1 second
- Edge ports (PortFast): Immediate
- Shared media: 3 seconds maximum

### Factors Affecting Convergence

**Network Factors:**
- Physical link detection speed
- Switch processing capability
- Network diameter (hop count)
- Number of VLANs

**Configuration Factors:**
- PortFast configuration
- Link type detection
- BPDU transmission intervals
- Path cost values

## Best Practices Implemented

### Design Principles
1. **Hierarchical Design**: Clear root bridge hierarchy
2. **Redundant Paths**: Multiple paths with automatic failover
3. **Fast Convergence**: PortFast and edge port optimization
4. **Security**: BPDU Guard and unused port protection

### Configuration Standards
1. **Consistent Priorities**: Predictable root bridge selection
2. **PortFast Deployment**: All access ports configured
3. **BPDU Guard**: Automatic protection on access ports
4. **Documentation**: Clear topology documentation

### Monitoring and Maintenance
1. **Regular Verification**: STP topology checks
2. **Event Monitoring**: STP change notifications
3. **Performance Monitoring**: Convergence time measurement
4. **Security Monitoring**: BPDU Guard violations

## Benefits

1. **Loop Prevention**: Eliminates Layer 2 loops
2. **Redundancy**: Provides backup paths for failures
3. **Fast Convergence**: Quick recovery from failures
4. **VLAN Support**: Per-VLAN optimization
5. **Security**: Protection against malicious switches
6. **Performance**: Optimized traffic paths
7. **Scalability**: Supports large switched networks

## Integration with Other Protocols

### HSRP Integration
- RSTP ensures Layer 2 connectivity
- HSRP provides Layer 3 redundancy
- Both protocols work together for end-to-end redundancy

### VLAN Integration
- Rapid PVST+ provides per-VLAN spanning tree
- Load balancing across different root bridges
- VLAN-specific path optimization

### EtherChannel Integration
- Reduces STP complexity
- Provides load balancing
- Improves convergence times