# Syslog and SNMP

Documentation about Syslog and SNMP configuration in Cisco devices for network monitoring and management.

## Overview

Syslog and SNMP are implemented across all network devices to provide centralized logging, monitoring, and management capabilities. LibreNMS is used as the central monitoring platform, collecting both syslog messages and SNMP data from all network devices.

## SNMPv2

Simple Network Management Protocol (SNMP) is a standard protocol for managing and monitoring network devices, such as routers, switches, servers, and printers. SNMP works by sending messages, called protocol data units (PDUs), to different parts of a network. SNMPv2 is an updated version of SNMPv1 and adds additional features.

SNMP was configured in every network device to allow monitoring with LibreNMS.

The SNMP-ACL is an ACL that allows only the LibreNMS server to access the SNMP service.

## LibreNMS

LibreNMS is an open-source network monitoring system that provides a web interface for monitoring network devices. It is capable of monitoring a wide range of devices, including routers, switches, servers, and printers. LibreNMS can be used to monitor network performance, detect network outages, and generate reports on network activity.

LibreNMS was installed in a PC running a VM with Ubuntu Server 22.04. The installation was done following the official documentation.

The LibreNMS server was configured to monitor all network devices using SNMPv2. The SNMP community string "cisco" was used to authenticate the devices.

## Syslog

Syslog is a standard for message logging. It allows separation of the software that generates messages, the system that stores them, and the software that reports and analyzes them.

## SNMP Configuration 

### Community String Configuration

All devices were configured with community string "cisco" in read-only mode.

```bash
snmp-server community cisco RO SNMP-ACL 
```

### SNMP Access Control

**SNMP-ACL Configuration:**
```bash
ip access-list standard SNMP-ACL
 permit 172.20.7.40             ! LibreNMS server
 deny   any
```

**Security Benefits:**
- **Restricted Access**: Only LibreNMS server can query SNMP
- **Read-Only Access**: Prevents unauthorized configuration changes
- **Network Security**: Blocks SNMP access from unauthorized sources
- **Monitoring Focus**: Dedicated access for monitoring purposes

### Additional SNMP Configuration

**Contact and Location Information:**
```bash
snmp-server contact network-admin@isepacademy.ccna.itn.com
snmp-server location "Zagreb Network Operations Center"
snmp-server chassis-id "RT1-Zg Main Router"
```

**SNMP System Information:**
```bash
snmp-server system-shutdown
snmp-server trap-source GigabitEthernet0/2
snmp-server enable traps snmp authentication linkdown linkup coldstart warmstart
```

## Syslog Configuration

### Syslog Server Configuration

All devices were configured to send syslog messages to the LibreNMS server.

```bash
logging facility local6
logging host 172.20.7.40
```

### Syslog Severity Levels

The syslog messages were configured to only be sent to the server if they are of severity level 6 (Notification) or higher.

**Syslog Severity Levels:**
- **0 - Emergency**: System is unusable
- **1 - Alert**: Action must be taken immediately
- **2 - Critical**: Critical conditions
- **3 - Error**: Error conditions
- **4 - Warning**: Warning conditions
- **5 - Notice**: Normal but significant condition
- **6 - Informational**: Informational messages
- **7 - Debug**: Debug-level messages

### Advanced Syslog Configuration

**Buffered Logging:**
```bash
logging buffered 16384
logging console critical
logging monitor informational
```

**Timestamps and Source Interface:**
```bash
service timestamps log datetime msec
service timestamps debug datetime msec
logging source-interface GigabitEthernet0/2
```

**Facility Configuration:**
```bash
logging facility local6
logging trap informational
logging host 172.20.7.40
```

## LibreNMS Server Configuration

### Server Details

**LibreNMS Server Specifications:**
- **IP Address**: 172.20.7.40
- **Operating System**: Ubuntu Server 22.04 LTS
- **Platform**: VMware Virtual Machine
- **Network Location**: Zagreb Management VLAN

### Monitored Parameters

**Device Information:**
- System uptime and availability
- CPU utilization and memory usage
- Interface statistics and utilization
- Environmental sensors (temperature, power)

**Network Performance:**
- Interface bandwidth utilization
- Packet loss and error rates
- Response time and latency
- SNMP walk and polling statistics

**Protocol-Specific Monitoring:**
- OSPF neighbor states and LSA information
- HSRP status and failover events
- IPSec tunnel status and statistics
- VoIP call quality and statistics

## Device Categories in LibreNMS

### Network Infrastructure

**Routers:**
- RT1-Zg (172.20.7.105) - Zagreb main router
- RT1-Pl (172.20.11.105) - Pula site router
- RT1-St (172.20.13.233) - Split site router

**Switches:**
- SW1-Zg (172.20.7.1) - Zagreb access switch
- SW2-Zg (172.20.7.2) - Zagreb access switch
- MLS1-Zg (172.20.7.3) - Zagreb core switch
- MLS2-Zg (172.20.7.4) - Zagreb core switch

**Servers:**
- DHCP-SRV (172.20.7.36) - DHCP and call manager server

## Monitoring Dashboards

### LibreNMS Dashboards

**Network Overview Dashboard:**
- Network topology and device status
- Critical alerts and notifications
- Bandwidth utilization summaries
- Availability statistics

**Device-Specific Dashboards:**
- Per-device interface graphs
- CPU and memory utilization
- Environmental monitoring
- Custom performance metrics

### Alert Configuration

**Critical Alerts:**
- Device down notifications
- Interface link failures
- High CPU/memory utilization
- IPSec tunnel failures

**Warning Alerts:**
- High interface utilization
- OSPF neighbor changes
- HSRP state changes
- Unusual traffic patterns

## Verification Commands

### SNMP Verification

**Test SNMP Connectivity:**
```bash
snmpwalk -v2c -c cisco 172.20.7.105 1.3.6.1.2.1.1.1.0
show snmp
show snmp community
```

**Check SNMP Statistics:**
```bash
show snmp statistics
show snmp engineID
show snmp group
```

### Syslog Verification

**Check Syslog Configuration:**
```bash
show logging
show logging history
show logging buffered
```

**Test Syslog Functionality:**
```bash
send log "Test message from router"
clear logging
```

### LibreNMS Verification

**From LibreNMS Server:**
```bash
./validate.php
./discovery.php -h all
./poller.php -h all
```

## Troubleshooting

### Common SNMP Issues

**SNMP Not Responding:**
1. Check ACL configuration
2. Verify community string
3. Test network connectivity
4. Check SNMP service status

**Debug SNMP:**
```bash
debug snmp packet
debug snmp detail
```

### Common Syslog Issues

**Messages Not Reaching Server:**
1. Check network connectivity
2. Verify syslog facility configuration
3. Check firewall rules
4. Verify server configuration

**Debug Syslog:**
```bash
debug logging
show logging statistics
```

## Security Considerations

### SNMP Security

**Community String Protection:**
- Use non-default community strings
- Implement ACL-based access control
- Regular community string rotation
- Monitor SNMP access attempts

### Syslog Security

**Message Integrity:**
- Secure syslog transmission (optional TLS)
- Centralized log storage
- Log retention policies
- Access control to log files

## Best Practices Implemented

### Monitoring Strategy
1. **Comprehensive Coverage**: All devices monitored
2. **Proactive Alerting**: Early warning systems
3. **Performance Baselines**: Historical data collection
4. **Capacity Planning**: Growth trend analysis

### Security Implementation
1. **Restricted Access**: ACL-based SNMP control
2. **Read-Only SNMP**: Prevents unauthorized changes
3. **Centralized Logging**: Secure log aggregation
4. **Access Auditing**: Monitor access to monitoring systems

### Operational Procedures
1. **Regular Reviews**: Weekly monitoring data analysis
2. **Alert Tuning**: Minimize false positives
3. **Documentation**: Clear escalation procedures
4. **Training**: Staff training on monitoring tools

## Benefits

1. **Network Visibility**: Comprehensive network monitoring
2. **Proactive Management**: Early problem detection
3. **Performance Optimization**: Data-driven improvements
4. **Security Monitoring**: Centralized security event tracking
5. **Compliance**: Audit trail and reporting capabilities
6. **Troubleshooting**: Faster problem resolution
7. **Capacity Planning**: Informed infrastructure decisions
8. **Cost Optimization**: Efficient resource utilization

## Integration with Network Services

### OSPF Integration
- Monitor neighbor relationships
- Track LSA propagation
- Detect routing changes
- Performance impact analysis

### HSRP Integration
- Monitor failover events
- Track active/standby states
- Alert on configuration changes
- Performance impact measurement

### IPSec Integration
- Monitor tunnel status
- Track encryption statistics
- Alert on connection failures
- Performance monitoring
