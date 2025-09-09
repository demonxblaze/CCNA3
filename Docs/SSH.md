# Secure Shell (SSH) Implementation

Documentation about the SSH configuration for secure remote management access.

## Overview

Secure Shell (SSH) is implemented across all network devices to provide encrypted remote management access. SSH has been configured to replace Telnet entirely, ensuring all remote management communications are encrypted and secure.

## SSH Security Benefits

1. **Encryption**: All communication is encrypted
2. **Authentication**: Strong authentication mechanisms
3. **Integrity**: Data integrity verification
4. **Key-based Authentication**: Public/private key authentication support
5. **Session Security**: Protection against session hijacking

## Global SSH Configuration

### Domain Name Configuration
SSH requires a domain name to generate RSA keys:

```bash
ip domain name isepacademy.ccna.itn.com
```

### User Account Configuration
Local user accounts with privilege 15 (full administrative access):

```bash
username cisco privilege 15 secret 5 $1$mERr$UIEd/gKB8Xu9v11aSkZ1n0
```

### RSA Key Generation
SSH requires RSA keys for encryption (typically generated with):

```bash
crypto key generate rsa general-keys modulus 2048
```

### IP SSH Configuration
```bash
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3
```

## VTY Line Configuration

SSH is configured on all VTY (Virtual Terminal) lines for remote access:

### VTY Lines 0-4
```bash
line vty 0 4
 access-class SSH-ACL in
 exec-timeout 0 30
 login local
 transport input ssh
```

### VTY Lines 5-15
```bash
line vty 5 15
 access-class SSH-ACL in
 exec-timeout 0 30
 login local
 transport input ssh
```

## SSH Access Control

### Access Control List (SSH-ACL)
SSH access is restricted using a standard ACL that permits only authorized management networks:

```bash
ip access-list standard SSH-ACL
 permit 172.20.7.108 0.0.0.3    ! MLS1-Zg and MLS2-Zg management
 permit 172.20.7.104 0.0.0.3    ! Router management range
 permit 10.0.0.0 0.0.0.3        ! Zagreb-Pula tunnel
 permit 10.0.0.4 0.0.0.3        ! Zagreb-Split tunnel
 permit 10.0.0.8 0.0.0.3        ! Pula-Split tunnel
 permit 172.20.11.104 0.0.0.7   ! Pula site management
 permit 172.20.13.232 0.0.0.7   ! Split site management
 permit 172.20.7.0 0.0.0.31     ! Zagreb management VLAN
 permit 172.20.10.128 0.0.0.127 ! Pula staff network
 permit 172.20.4.0 0.0.0.255    ! Zagreb administration network
 permit 172.20.5.0 0.0.0.255    ! Zagreb staff network
 permit 172.20.12.0 0.0.1.255   ! Split networks
 deny   any
```

### Application to VTY Lines
```bash
line vty 0 15
 access-class SSH-ACL in
```

## Authentication Configuration

### Local Authentication
```bash
line vty 0 15
 login local
```

This configuration requires users to authenticate using locally configured usernames and passwords.

### Privilege Levels
```bash
username cisco privilege 15 secret [password]
```

Privilege 15 provides full administrative access to the device.

## Session Security Settings

### Session Timeout
```bash
line vty 0 15
 exec-timeout 0 30
```

Sessions timeout after 30 minutes of inactivity (0 hours, 30 minutes).

### Login Block
```bash
login block-for 300 attempts 3 within 120
```

- **Block Duration**: 300 seconds (5 minutes)
- **Failed Attempts**: 3 attempts
- **Time Window**: 120 seconds (2 minutes)

This prevents brute force attacks by blocking SSH access after 3 failed login attempts within 2 minutes.

## SSH Transport Security

### Transport Input Restriction
```bash
line vty 0 15
 transport input ssh
```

This configuration:
- **Enables**: SSH protocol only
- **Disables**: Telnet, rlogin, and other insecure protocols
- **Forces**: Encrypted connections only

### Supported SSH Versions
```bash
ip ssh version 2
```

Restricts SSH to version 2 only, which is more secure than version 1.

## SSH Connection Parameters

### Connection Timeout
```bash
ip ssh time-out 60
```

SSH connections timeout after 60 seconds if authentication is not completed.

### Authentication Retries
```bash
ip ssh authentication-retries 3
```

Allows maximum of 3 authentication attempts per connection.

## Host Key Verification

### Known Hosts Configuration
Static host entries for known devices:

```bash
ip host MLS1-Zg 172.20.7.106
ip host MLS2-Zg 172.20.7.110
ip host RT1-Pl 172.20.11.105
ip host RT1-St 172.20.13.233
ip host RT1-Zg 172.20.7.105
ip host SW1-Pl 172.20.11.106
ip host SW1-St 172.20.13.234
ip host SW1-Zg 172.20.7.1
ip host SW2-Zg 172.20.7.2
ip host DHCP-SRV 172.20.7.36
```

Benefits:
- **Name Resolution**: Use hostnames instead of IP addresses
- **Security**: Verify connection to correct devices
- **Convenience**: Easier device identification

## Management Access Architecture

### Centralized Management
SSH access is designed for centralized management from:
- **Management VLAN**: 172.20.7.0/27
- **Administration Network**: 172.20.4.0/24
- **Staff Networks**: Various site-specific networks
- **Inter-site Tunnels**: IPSec tunnel endpoints

### Multi-site Access
SSH access is configured to work across all three sites:
- **Zagreb**: Direct LAN access and tunnel endpoints
- **Pula**: Access via IPSec tunnel and local management
- **Split**: Access via IPSec tunnel and local management

## Verification Commands

### SSH Status
```bash
show ip ssh
show ssh
```

### Active SSH Sessions
```bash
show users
show line vty
```

### SSH Connection Statistics
```bash
show ip ssh statistics
```

### Test SSH Connectivity
```bash
ssh -l [username] [ip-address]
```

## Security Best Practices Implemented

### Access Control
1. **ACL Filtering**: Source-based access control
2. **VTY Restrictions**: SSH-only transport
3. **Privilege Separation**: Role-based access
4. **Session Limits**: Timeout and retry limits

### Authentication
1. **Strong Passwords**: Encrypted password storage
2. **Local Authentication**: No external dependencies
3. **Privilege Levels**: Administrative access control
4. **Brute Force Protection**: Login blocking

### Protocol Security
1. **SSH Version 2**: Latest SSH protocol
2. **Strong Encryption**: RSA key-based encryption
3. **No Telnet**: Insecure protocols disabled
4. **Transport Security**: Encrypted sessions only

## Troubleshooting SSH

### Common Issues
1. **RSA Keys Missing**: Regenerate crypto keys
2. **Domain Name Missing**: Configure ip domain-name
3. **ACL Blocking**: Check SSH-ACL configuration
4. **Authentication Failures**: Verify local user accounts

### Debug Commands
```bash
debug ip ssh
debug ssh
```

### SSH Client Commands
```bash
ssh -v [hostname]    ! Verbose mode
ssh -l [user] [host] ! Specify username
```

## Benefits

1. **Security**: Encrypted management communications
2. **Authentication**: Strong user authentication
3. **Access Control**: Granular access restrictions
4. **Monitoring**: Session logging and tracking
5. **Compliance**: Meets security standards
6. **Remote Management**: Secure multi-site administration