# Single Area OSPF Implementation

Documentation of the implementation of the Single Area OSPF Operation

## Configuration

In every single router OSPF was configured with process ID 100.

```bash
router ospf 100
```

To estabilish the routes between sites, OSPF was configured to propagate the IPSec Tunnels interface and the site's networks.

### Zagreb Router:

```bash
 network 10.0.0.1 0.0.0.0 area 0
 network 10.0.0.5 0.0.0.0 area 0
 network 172.20.7.105 0.0.0.0 area 0
 network 172.20.7.109 0.0.0.0 area 0
 default-information originate
```

### Split Router:

```bash
 network 10.0.0.10 0.0.0.0 area 0
 network 10.0.0.6 0.0.0.0 area 0
 network 172.20.12.0 0.0.1.255 area 0
 default-information originate
```

### Pula Router:

```bash
 network 10.0.0.2 0.0.0.0 area 0
 network 10.0.0.9 0.0.0.0 area 0
 network 172.20.9.0 0.0.3.255 area 0
 default-information originate
```

### Zagreb MLS's:

To provide redundancy in the network, and intervlan routing, the MLS's were also configured with OSPF.

```bash
 network 172.20.7.106 0.0.0.0 area 0
 network 172.20.0.0 0.0.7.255 area 0
```

### Default Route

Since the router's default gateway's IP is configured via DHCP, the default route was configured as:

```bash
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/2
```

The route was propagated to the OSPF process with the command:

```bash
default-information originate
```

## Verification

To verify the OSPF configuration, the following commands were used:

```bash
show ip ospf neighbor
show ip ospf interface
show ip route
```

The output of the commands was analyzed to verify if the OSPF process was working correctly and if the routes were being propagated correctly.
