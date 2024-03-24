# NAT basics
## Reference
https://community.cisco.com/t5/service-providers-knowledge-base/cgv6-on-ism-cgn-nat44-deployment-guide/ta-p/3153058/page/4
https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/217208-understanding-nat64-and-its-configuratio.html

## Table of Contents
-   [NAT44](#nat44)
-   [NAT64](#nat64)
## NAT44
Private source Ip address and port number is translated to Public IP address and port number.
An entry is created in the NAT DB for this translation.
For TCP/UDP packets, the checksum field in the TCP/UDP header has to be updated as the checksum calculation depends on source address and port also.

### Implementation summary
-  The NAT44 device will have a set of interfaces connecting to the private network, and a set of interfaces connecting to the public network.
-  NAT44 Private interfaces will belong to a separate VRF (iVRF), and Public interfaces will belong to a separate VRF (oVRF)
-  Think of VRF as an instance of a routing table. So there will be two set of routing tables.
#### Private to public flow
-  To deliver the packet from the private instance to the NAT44 instance for translation, a virtual interface is created -say vInt1 in iVRF. A static route is added in the
   iVRF to forward all private traffic to vInt1. This can also be achieved by Access-List Based Forwarding, where an access list iACL is created and applied to each
   private interface. This vInt1 is assigned to the NAT44 instance
-  After the packet from the private instance reaches the NAT44 card, NAT44 replaces the source address and source port with a translated address and port. An entry
   is created in the NAT44 DB for this translation and the packet is sent out on the public interface by resolving the destination address lookup on the oVRF FIB.
#### Public to private flow
-  To take the public packet to the NAT44 instance, a virtual interface is created in the oVRF --say vInt2.
-  A static route is added in the oVRF to forward all public traffic to vInt2. This can also be achieved by ABF. THe vInt2 is assigned to the NAT44 service instance.
-  After the packet from public network reaches the NAT44 instance, the NAT44 DB is checked for an entry for the destination address. If found, the destination address/port is replaced with the private address/port and then forward via the private interface which is found out by destination address lookup on iVRF.
-  The Checksum in the TCP/UDP header is updated before sending the packet out.


![Nat44](https://github.com/ansumanm/Docs/blob/master/nat44.png "Nat44")

## NAT64
Together with DNS64, the primary purpose if NAT64 is to allow an IPv6 only client to initiate communications to an IPV4 only server. NAT64 can also be used for IPv4 only clients initiating communications with IPv6-only servers using static or manual bindings. Both source and destination v6 addresses are translated to v4 addresses.

### Stateless NAT64
In stateless NAT64, state is not preserved which means for every IPv6 user a dedicated IPv4 address is required.

### Stateful NAT64
In stateful NAT64, states are maintained. A single IP address is used for all teh private users with different port numbers. 

## Main components of NAT64
### Nat64 prefix
Any /32. /40, /48, /56, /64 or /96 prefix used with a converted IPv4 address to transmit the packet over the v6 only network. 

### DNS64 server: 
The DNS64 server functions as a normal DNS server for IPv6 AAAA records but can also attempt to locate an IPv4 A record when a AAAA record is not available. If an A record is located, DNS64 converts the IPv4 A record into an IPv6 AAAA record using the NAT64 prefix. This gives the impression to the IPv6-only host that it can communicate with a server using IPv6.

### NAT64 router:
The NAT64 router advertises the NAT64 prefix into the IPv6-only network along with performing the translation between the IPv6-only and IPv4-only networks.
