# DHCP implementation in the network

Documentation about the implementation of the DHCP service in the network.

## Configuration

The routers in Split and Pula were configured to provide DHCP service to the network. In the Zagreb site, was provided by a router that we used as a DHCP server and call manager.

The configuration was done with the following guidelines:

- Each VLAN was configured with a DHCP pool
- The default router was configured with the VLAN corresponding router subinterface's IP address
- The domain name was configured with isepacademy.ccna.itn.com
- The DNS servers configured were ISEP Academy DNS (193.136.60.10 and 193.136.60.2)
- The VOIP pools were configured with a lease time of 40 and with option 150 (TFTP server IP address) configured with the IP address of the router that was used as a call manager (Zagreb DHCP Server, Split Router and Pula Router)
