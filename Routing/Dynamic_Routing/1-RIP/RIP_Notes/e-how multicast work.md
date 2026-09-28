**How multicast works — using RIPv2**

**Multicast means sending a packet to a group of interested receivers.** The sender uses a multicast group address instead of addressing each receiver separately.

For RIPv2, that group address is **`224.0.0.9`**.

**1. Example network**

Assume R1, R2, PC1 and PC2 connect to the same switch in the same VLAN.

|Device|Interface IP|RIPv2 enabled?|
|---|---|---|
|R1|`192.168.1.1/24`|Yes|
|R2|`192.168.1.2/24`|Yes|
|PC1|`192.168.1.10/24`|No|
|PC2|`192.168.1.11/24`|No|

R1 wants to send a routing update that R2 can use.

**2. R2 prepares to receive multicast**

When RIPv2 is enabled on R2’s interface, its software tells the network stack to listen for **`224.0.0.9`** on that interface.

R2 now recognizes both these destinations for local reception:

|Destination|Why R2 accepts it|
|---|---|
|`192.168.1.2`|Its own interface IP address|
|`224.0.0.9`|A multicast group it listens to|

**The multicast address does not replace R2’s interface IP.** It is an additional group destination that R2 recognizes. Multiple routers can listen to the same group.

Joining a group means recording that interest in the device’s networking system. It does not mean asking R1 for permission.

**3. R1 creates the update**

On this Ethernet LAN, R1 creates a packet and frame with these addresses:

|Field|Value|
|---|---|
|Source IP|`192.168.1.1`|
|Destination IP|`224.0.0.9`|
|Destination Ethernet MAC|`01:00:5E:00:00:09`|
|Transport protocol|UDP|
|Destination UDP port|`520`|

The multicast destination MAC is derived from the multicast IP address. **R1 does not use ARP to find an individual router’s MAC for this multicast destination.**

R1 does not need a list of receivers. It sends the update to the group address through its RIP-enabled, non-passive interface.

**4. The switch forwards the frame**

For RIPv2’s local multicast address, the switch normally forwards the frame through the other forwarding ports in the same VLAN.

Therefore, the frame can reach:

- R2.
    
- PC1.
    
- PC2.
    

It is not sent back through the port where it arrived.

**In this example, multicast does not reduce the number of switch ports receiving the frame compared with broadcast.** The difference appears when the receiving devices filter it.

**5. R2 accepts and processes it**

R2’s receiving process is:

1. Its networking system recognizes the multicast destination.
    
2. Its IP stack accepts `224.0.0.9` because R2 listens to that group.
    
3. UDP delivers the data to the RIP process listening on port `520`.
    
4. RIP checks the update and uses valid routing information when appropriate.
    

Accepting the multicast destination is only the first step. It does not mean every advertised route will automatically be installed.

**6. The PCs discard it**

PC1 and PC2 are not listening to the RIPv2 group.

A PC can reject this traffic at either of these stages:

|Stage|What happens|
|---|---|
|Network card filtering|The card can reject the unwanted multicast destination MAC before passing the packet to the operating system.|
|IP processing, if the frame passes the card|The networking software sees that the PC has not joined `224.0.0.9` and discards the packet.|

The exact filtering behavior depends on the network card and operating system.

**Receiving a frame on the cable does not mean delivering its contents to an application.**

**7. Why this differs from broadcast**

Assume a normal PC is not running RIP, with no firewall rule dropping the traffic earlier:

|Processing step|RIPv1 broadcast|RIPv2 multicast|
|---|---|---|
|Frame reaches the PC’s cable|Yes|Can happen|
|Destination MAC check|Broadcast MAC is accepted|Unwanted multicast MAC can be filtered|
|Destination IP check|`255.255.255.255` includes the PC|`224.0.0.9` is not a group the PC joined|
|If it reaches UDP|No application listening on port `520` → discard|Can already have been discarded|
|PC processes RIP routes|No|No|

==With broadcast, the PC cannot reject the packet simply because the destination is “not mine”: **broadcast includes the PC**. It can later discard the packet because no application is listening on the destination port.==

With multicast, **the group address itself provides a reason to reject the packet earlier**.

**8. Multicast and passive interfaces**

These control different things:

|Feature|Meaning|
|---|---|
|Multicast|Send an update addressed to a particular group.|
|Passive interface|Do not send RIP updates through this interface.|

If R2 and the PCs share the same LAN, making R1’s LAN interface passive stops updates to **R2 as well**.

Multicast allows R1 to keep sending updates for R2 while uninterested PCs can filter them.

**Remember:** The switch behavior described here is specifically for RIPv2’s local multicast group. For other multicast groups, a switch using IGMP snooping can restrict forwarding to interested ports. Multicast does not always mean flooding every port.