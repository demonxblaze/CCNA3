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
