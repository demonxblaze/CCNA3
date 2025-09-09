# IPSec Tunnel configuration

Documentation about the IPSec Tunnel configuration between sites.

## Overview

To estabilish a secure connection between the sites, IPSec tunnels were configured between the routers in Split and Pula and the Zagreb router.
The tunnels were configured to use the IKEv1 protocol and the ESP protocol for encryption.

## Configuration

Configuration was done with the following guidelines:

- Pula | Split tunnel **10.0.0.8/30**
  - Pula: 10.0.0.9
  - Split: 10.0.0.10

- Pula | Zagreb Tunnel **10.0.0.0/30**
  - Pula: 10.0.0.2
  - Zagreb: 10.0.0.1

- Split | Zagreb Tunnel **10.0.0.4/30**
  - Split: 10.0.0.6
  - Zagreb: 10.0.0.5

- Phase 1:
  - Policy number 100
  - Encryption: AES128
  - Hash: SHA
  - Authentication: Pre-shared key
  - DH Group: 5

```bash
crypto isakmp policy 100
 encr aes
 authentication pre-share
 group 5
 hash sha
 ```

- Pre-shared key:
  - Zagreb and Pula – z4gr3bPvl4
  - Zagreb and Split - z4gr3b$pl1t
  - Pula and Split - Pvl4$pl1t

```bash
crypto isakmp key z4gr3bPvl4 address 0.0.0.0  
crypto isakmp key z4gr3b$pl1t address 0.0.0.0
crypto isakmp key Pvl4$pl1t address 0.0.0.0
```

- Phase 2:
  - IPsec transform set: TSET
  - Security Protocol: ESP
  - Encryption: AES128
  - Hash: SHA

```bash
crypto ipsec transform-set TSET esp-3des esp-sha-hmac 
 mode tunnel
```

- An ACL was created to match the traffic that should be encrypted:

```bash
ip access-list extended CRYPTO ACL
 permit gre host ipsecTunnelIP host ipsecTunnelIP
 deny ip any any
```

The configuration of the tunnels was done with the following commands:

```bash
interface Tunnel0
 description Description of the tunnel
 ip address ip mask
 tunnel source GigabitEthernet
 tunnel destination remoteInterfaceIP
!
interface GigabitEthernet0/2
 description Connection to IsepAcademy Internet
 ip address ip mask
 ip nat outside
 ip virtual-reassembly in
 ipv6 address FE80::1:1 link-local
 crypto map CROATIA
!
```

## Verification Commands

### IPSec Status Verification

**Check IKE Phase 1 (ISAKMP) Status:**
```bash
show crypto isakmp sa
show crypto isakmp policy
show crypto isakmp key
```

**Check IPSec Phase 2 Status:**
```bash
show crypto ipsec sa
show crypto ipsec transform-set
show crypto map
```

### Tunnel Interface Verification

**Check Tunnel Status:**
```bash
show interface tunnel 0
show interface tunnel 30
show ip interface brief | include Tunnel
```

**Check Tunnel Routing:**
```bash
show ip route | include Tunnel
ping [remote-tunnel-ip]
traceroute [remote-network]
```

### Traffic and Statistics

**IPSec Traffic Statistics:**
```bash
show crypto ipsec sa detail
show crypto ipsec transform-set
show crypto engine connections active
```

**Tunnel Traffic Verification:**
```bash
show interface tunnel 0 | include packets
show ip traffic | include Tunnel
```

## Troubleshooting Commands

### Debug IPSec Issues

**IKE Phase 1 Debugging:**
```bash
debug crypto isakmp
debug crypto isakmp sa
debug crypto isakmp error
```

**IPSec Phase 2 Debugging:**
```bash
debug crypto ipsec
debug crypto ipsec sa
debug crypto ipsec error
```

### Common Verification Steps

**1. Check Crypto Map Application:**
```bash
show crypto map interface
show crypto map tag CROATIA
```

**2. Verify Interesting Traffic:**
```bash
show access-lists CRYPTO-ACL-PULA
show access-lists CRYPTO-ACL-SPLIT
```

**3. Check Reachability:**
```bash
ping 10.0.0.2    ! Zagreb to Pula tunnel
ping 10.0.0.6    ! Zagreb to Split tunnel
ping 10.0.0.10   ! Pula to Split tunnel
```

## Expected Output Examples

### Successful ISAKMP SA
```
Router# show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
172.16.201.77   172.16.201.80   QM_IDLE           1001 ACTIVE
172.16.201.79   172.16.201.80   QM_IDLE           1002 ACTIVE
```

### Active IPSec SA
```
Router# show crypto ipsec sa

interface: GigabitEthernet0/2
    Crypto map tag: CROATIA, local addr 172.16.201.80

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (172.16.201.80/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (172.16.201.77/255.255.255.255/47/0)
   current_peer 172.16.201.77 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 1234, #pkts encrypt: 1234, #pkts digest: 1234
    #pkts decaps: 5678, #pkts decrypt: 5678, #pkts verify: 5678
```

## Security Best Practices

### Key Management
- Regular key rotation recommended
- Strong pre-shared keys used
- Keys documented securely
- Different keys per tunnel pair

### Monitoring and Maintenance
- Regular SA status verification
- Traffic pattern monitoring
- Performance baseline establishment
- Security event logging

### Backup Procedures
- Configuration backup before changes
- Tunnel failover testing
- Recovery procedures documented
- Emergency contact procedures
