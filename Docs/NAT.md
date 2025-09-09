# Network Address Translation (NAT)

Documentation about the implementation of NAT for internet connectivity in the Zagreb site.

## Overview

Network Address Translation (NAT) is configured on the Zagreb router (RT1-Zg) to provide internet connectivity for all three sites. The router performs both dynamic NAT with overload (PAT) for general internet access and static NAT for specific services that need to be accessible from the internet.

## NAT Configuration

### Interface Configuration

**Inside Interfaces (LAN-facing):**
```bash
interface GigabitEthernet0/0
 description Connection to MLS1-Zg
 ip address 172.20.7.105 255.255.255.252
 ip nat inside

interface GigabitEthernet0/1
 description Connection to MLS2-Zg
 ip address 172.20.7.109 255.255.255.252
 ip nat inside
```

**Outside Interface (Internet-facing):**
```bash
interface GigabitEthernet0/2
 description Connection to IsepAcademy Internet
 ip address 172.16.201.80 255.255.255.0
 ip nat outside
```

### Dynamic NAT with Overload (PAT)

Dynamic NAT with overload allows multiple internal devices to share a single public IP address by using different port numbers.

```bash
ip nat inside source list 1 interface GigabitEthernet0/2 overload
```

**Access List for NAT:**
```bash
access-list 1 permit 172.20.0.0 0.0.7.255
access-list 1 permit 172.20.7.108 0.0.0.3
access-list 1 permit 172.20.7.104 0.0.0.3
access-list 1 deny any
```

This configuration allows:
- All devices in the 172.20.0.0/21 network (covers all VLANs)
- Specific management IP ranges
- Inter-site tunnel traffic

### Static NAT Configuration

Static NAT is configured for the DHCP server to allow external access to specific services:

**SSH Access (Port 22):**
```bash
ip nat inside source static tcp 172.20.7.36 22 172.16.201.80 22 extendable
```

**HTTP Access (Port 80):**
```bash
ip nat inside source static tcp 172.20.7.36 80 172.16.201.80 80 extendable
```

**HTTPS Access (Port 443):**
```bash
ip nat inside source static tcp 172.20.7.36 443 172.16.201.80 443 extendable
```

These configurations map the DHCP server (172.20.7.36) services to the public IP (172.16.201.80) for external access.

## Access Control

### Inbound Access Control
```bash
interface GigabitEthernet0/0
 ip access-group 1 in
 ip access-group 101 out

interface GigabitEthernet0/1
 ip access-group 1 in
 ip access-group 101 out
```

These ACLs control:
- **Inbound (ACL 1)**: What traffic can enter from the LAN
- **Outbound (ACL 101)**: What traffic can exit to the LAN

## Internet Connectivity for Remote Sites

Remote sites (Pula and Split) access the internet through:
1. **OSPF routing** to reach the Zagreb router
2. **IPSec tunnels** for secure connectivity
3. **NAT translation** at the Zagreb router
4. **Default route** pointing to Zagreb

### Routing Configuration
```bash
ip route 0.0.0.0 0.0.0.0 172.16.201.254
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/2
```

Two default routes provide redundancy:
- Primary: Static route to ISP gateway
- Backup: Interface-based route

## NAT Pool vs Interface Overload

This implementation uses **interface overload** rather than a NAT pool because:
- Only one public IP address is available (172.16.201.80)
- More efficient for small to medium networks
- Automatic IP assignment from the interface
- Simpler configuration and management

## IP Address Allocation

### Private IP Ranges Used:
- **Zagreb VLANs**: 172.20.0.0/21 (172.20.0.0 - 172.20.7.255)
- **Pula Networks**: 172.20.8.0/22 (172.20.8.0 - 172.20.11.255)
- **Split Networks**: 172.20.12.0/22 (172.20.12.0 - 172.20.15.255)

### Public IP Address:
- **Internet Interface**: 172.16.201.80/24
- **ISP Gateway**: 172.16.201.254

## Security Considerations

### NAT Security Benefits:
1. **IP Address Hiding**: Internal structure not visible externally
2. **Implicit Firewall**: Inbound connections blocked by default
3. **Port-based Access**: Static NAT only for required services
4. **Access Control**: ACLs provide additional filtering

### Security Limitations:
- Some protocols may require special handling (VPN, multimedia)
- End-to-end connectivity challenges for certain applications
- Logging and troubleshooting complexity

## Verification Commands

**NAT Translation Table:**
```bash
show ip nat translations
show ip nat translations verbose
```

**NAT Statistics:**
```bash
show ip nat statistics
```

**Interface NAT Status:**
```bash
show ip interface [interface] | include NAT
```

**Clear NAT Translations:**
```bash
clear ip nat translation *
clear ip nat translation inside [local-ip] [global-ip]
```

**Debug NAT:**
```bash
debug ip nat
debug ip nat detailed
```

## Troubleshooting

### Common Issues:
1. **Missing inside/outside configuration** on interfaces
2. **Incorrect ACL matching** for NAT
3. **Conflicting static routes** with NAT
4. **Translation table overflow**

### Troubleshooting Steps:
1. Verify interface NAT configuration
2. Check access list permits required traffic
3. Verify translation table entries
4. Test connectivity from inside to outside
5. Monitor NAT statistics for errors

## Benefits

1. **IP Address Conservation**: Multiple devices share single public IP
2. **Security**: Hidden internal network structure
3. **Flexibility**: Easy internal IP changes without affecting external connectivity
4. **Cost Savings**: Reduced need for multiple public IP addresses
5. **Scalability**: Supports thousands of internal devices