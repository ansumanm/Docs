# NAT basics
## Reference
https://community.cisco.com/t5/service-providers-knowledge-base/cgv6-on-ism-cgn-nat44-deployment-guide/ta-p/3153058/page/4
https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/217208-understanding-nat64-and-its-configuratio.html

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
-  
![Nat44](https://github.com/ansumanm/Docs/blob/master/nat44.png "Nat44")
