# Syslog and SNMP

Documentation about Syslog and SNMP configuration in Cisco devices.

## SNMPv2

Simple Network Management Protocol (SNMP) is a standard protocol for managing and monitoring network devices, such as routers, switches, servers, and printers. SNMP works by sending messages, called protocol data units (PDUs), to different parts of a network. SNMPv2 is an updated version of SNMPv1 and adds additional features.

SNMP was configured in every network device to allow monitoring with LibreNMS.

### Configuration

All devices were configured with community string "cisco" in read-only mode.


```bash	
snmp-server community cisco RO SNMP-ACL 
```

The SNMP-ACL is an ACL that allows only the LibreNMS server to access the SNMP service.

### LibreNMS

LibreNMS is an open-source network monitoring system that provides a web interface for monitoring network devices. It is capable of monitoring a wide range of devices, including routers, switches, servers, and printers. LibreNMS can be used to monitor network performance, detect network outages, and generate reports on network activity.

LibreNMS was installed in a PC running a VM with Ubuntu Server 22. The installation was done following the official documentation.

The LibreNMS server was configured to monitor all network devices using SNMPv2. The SNMP community string "cisco" was used to authenticate the devices.

## Syslog

Syslog is a standard for message logging. It allows separation of the software that generates messages, the system that stores them, and the software that reports and analyzes them.

### Configuration

All devices were configured to send syslog messages to the LibreNMS server.

The syslog messages were configured to only be sent to the server if they are of severity level 6 (Notification) or higher.

```bash
logging facility local6
logging host 172.20.7.40
```