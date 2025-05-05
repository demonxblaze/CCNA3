# VoIP

Documentation about VoIP configuration in Cisco devices.

## VoIP Overview

Voice over Internet Protocol (VoIP) is a technology that allows users to make voice calls using a broadband Internet connection instead of a regular (or analog) phone line. VoIP works by converting sound into digital voice communication and then transferring it through the Internet. 

In this solution we set up a VoIP network using Cisco IP Phones and used routers (Split & Pula) and Servers (Zagreb DHCP Server) as Call Managers.

## Configuration

### IP Phones

The IP Phones were configured with the following settings:

- Dial Number: X00N
  - X - Location (1 - Zagreb , 2 - Pula, 4 - Split)
  - N - Extension Number

- Call Manager:
  - Zagreb DHCP Server (Zagreb Phones)
  - Split Router (Split Phones)
  - Pula Router (Pula Phones)
  - Vlan 40 (Voice Vlan)
  - Default Gateway: Vlan 40 IP Address

Configuration was done following Annex 2 in this [file](<../CCNAv7 ENSA Final Lab-2023.pdf>)
