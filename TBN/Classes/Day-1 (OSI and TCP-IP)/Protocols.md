
## 1. What is a protocol?

***A protocol is a set of rules that defines how devices communicate and exchange data over a network.***

These rules specify things such as ***how data is structured,*** ***how communication happens,*** and ***how errors are handled***.

**Examples:**

- **TCP**: Provides reliable, ordered data delivery.
- **IP**: Handles logical addressing and packet delivery between networks.
- **HTTP**: Defines how web clients and servers exchange web data.
- **DNS**: Defines how domain-name queries and responses are exchanged.

## 2. What is a port number?

A port number is a logical number used by TCP or UDP to ***identify a particular application or service on a device***.

It helps the operating system deliver incoming data to the correct application.

Examples:

- Port 80 — HTTP
- Port 443 — HTTPS
- Port 53 — DNS
- Port 22 — SSH

## 3. Difference between a protocol and a port number

| Feature          | Protocol                                                                  | Port number                                            |
| ---------------- | ------------------------------------------------------------------------- | ------------------------------------------------------ |
| Definition       | Rules governing communication.                                            | Number identifying an application or service endpoint. |
| Purpose          | Defines how communication or data exchange works.                         | Helps deliver data to the correct application.         |
| Example          | TCP, UDP, HTTP, DNS                                                       | TCP port 80, UDP port 53                               |
| Where it appears | Depends on the protocol; for example, TCP is identified in the IP header. | In the TCP or UDP header.                              |
| Example number   | TCP = IP protocol number 6                                                | HTTP commonly uses TCP port 80.                        |

## 4. Understand with an example

Imagine you're opening `https://example.com` in your browser.

Protocol: TCP

TCP defines how data is reliably delivered. Its IP protocol number is 6.

Port number: 443

The web server uses TCP port 443 for HTTPS connections by default.

Think of it this way: the protocol defines the rules of communication, while the port number identifies the destination service.

One important detail: HTTPS is an application-layer protocol, TCP is a transport-layer protocol, and 443 is a TCP port number. They serve different purposes but work together.




# Network Protocols by OSI Layer

In networking, protocols operate at different layers of the OSI model and TCP/IP model. Some protocols have an official IP Protocol Number, while others use TCP/UDP port numbers, Ethernet type values, or IEEE protocol identifiers.

Important: An IP protocol number and a port number are different. For example, TCP has IP protocol number 6, while HTTP commonly uses TCP port 80.

## 1. Application layer (OSI Layer 7)

Protocols used by applications and services to communicate over a network.

|Protocol|Port number (TCP/UDP)|Brief explanation|
|---|---|---|
|HTTP|TCP 80|Transfers web pages and web data.|
|HTTPS|TCP 443|Secure HTTP communication using TLS.|
|DNS|UDP/TCP 53|Resolves domain names to IP addresses and other DNS records.|
|DHCP|UDP 67, 68 (IPv4)|Automatically assigns IP configuration to clients.|
|FTP|TCP 20, 21|Transfers files; traditionally uses separate control and data connections.|
|SFTP|TCP 22|Secure file transfer over SSH.|
|SSH|TCP 22|Secure remote login and command execution.|
|Telnet|TCP 23|Remote terminal access without encryption by default.|
|SMTP|TCP 25, 587, 465|Sends email between mail systems or from clients.|
|POP3|TCP 110, 995|Retrieves email from a mail server.|
|IMAP|TCP 143, 993|Accesses and synchronizes email stored on a server.|
|SNMP|UDP 161, 162|Monitors and manages network devices.|
|NTP|UDP 123|Synchronizes system clocks.|
|TFTP|UDP 69|Simple file transfer, often used for network-device booting.|
|LDAP|TCP/UDP 389|Accesses directory services.|
|LDAPS|TCP 636|LDAP protected with TLS.|
|SIP|TCP/UDP 5060; commonly TLS 5061|Establishes and manages voice/video communication sessions.|
|RDP|TCP/UDP 3389|Remote desktop access, commonly used with Windows.|

Note: These are conventional/default ports; actual configurations can differ. DHCPv6 uses UDP ports 546 and 547.

## 2. Transport layer (OSI Layer 4)

Provides end-to-end communication between applications using ports, with either reliable delivery or lightweight datagrams.

|Protocol|IP protocol number|Brief explanation|
|---|---|---|
|TCP|6|Connection-oriented transport; provides reliable, ordered delivery.|
|UDP|17|Connectionless transport; low overhead, without built-in delivery guarantees.|
|SCTP|132|Message-oriented transport supporting multiple streams and multiple network paths.|
|DCCP|33|Provides congestion-controlled datagram transport.|

Remember: TCP and UDP port numbers identify applications or services. Their IP protocol numbers identify the transport protocol carried inside an IP packet.

## 3. Network layer (OSI Layer 3)

Responsible for logical addressing, routing and forwarding packets between networks.

|Protocol|IP protocol number|Brief explanation|
|---|---|---|
|IPv4|—|Provides IPv4 addressing and packet delivery. The IPv4 header's Protocol field identifies the next protocol.|
|IPv6|—|Provides IPv6 addressing and packet delivery. Its Next Header field identifies the following header or protocol.|
|ICMP|1|Reports IP delivery errors and supports diagnostics such as `ping` in IPv4.|
|ICMPv6|58|Error reporting and diagnostics for IPv6, including functions needed for neighbor discovery.|
|IGMP|2|Manages IPv4 multicast group membership.|
|OSPF|89|Exchanges routing information between routers within an autonomous system.|
|EIGRP|88|Cisco-developed routing protocol used to exchange routes.|
|RIP|UDP 520|Distance-vector routing protocol that uses hop count as its metric.|
|BGP|TCP 179|Exchanges routing information between autonomous systems.|
|GRE|47|Encapsulates network-layer packets inside GRE tunnels.|
|IPsec ESP|50|Provides IP packet confidentiality and/or integrity and authentication.|
|IPsec AH|51|Provides IP packet integrity and authentication, but not encryption.|
|PIM|103|Supports multicast routing.|
|VRRP|112|Provides default-gateway redundancy between routers.|
|IP-in-IP|4|Encapsulates one IP packet inside another IP packet.|

Important distinction: RIP and BGP use transport-layer ports, whereas OSPF and EIGRP are identified directly by IP protocol numbers. OSPF does not use TCP or UDP ports for its standard operation.

## 4. Data Link layer (OSI Layer 2)

Handles communication over a local link, including MAC addressing, Ethernet framing and VLAN tagging.

For Layer 2, identifiers such as EtherType are commonly used instead of IP protocol numbers.

|Protocol / standard|Identifier|Brief explanation|
|---|---|---|
|Ethernet (IPv4)|EtherType `0x0800`|Identifies IPv4 packets inside Ethernet frames.|
|Ethernet (IPv6)|EtherType `0x86DD`|Identifies IPv6 packets inside Ethernet frames.|
|ARP|EtherType `0x0806`|Resolves an IPv4 address to a link-layer address on a local network.|
|IEEE 802.1Q|EtherType `0x8100`|Adds VLAN tagging to Ethernet frames.|
|LLDP|EtherType `0x88CC`|Advertises device identity and capabilities to directly connected network devices.|
|PPPoE Discovery|EtherType `0x8863`|Discovers PPPoE access concentrators and establishes a session.|
|PPPoE Session|EtherType `0x8864`|Carries PPP traffic over Ethernet after session establishment.|
|STP|IEEE 802.1D / LLC|Prevents Layer 2 switching loops by building a loop-free topology.|
|LACP|EtherType `0x8809`|Negotiates link aggregation to combine physical links into a logical link.|

## 5. Physical layer (OSI Layer 1)

This layer transmits raw bits as electrical, optical or radio signals. It mainly involves transmission standards rather than IP protocols or port numbers.

|Standard / technology|Brief explanation|
|---|---|
|100BASE-TX|Ethernet over copper at 100 Mbps.|
|1000BASE-T|Gigabit Ethernet over copper at 1 Gbps.|
|10GBASE-SR|10-Gigabit Ethernet over short-range multimode fiber.|
|802.11 PHY|Defines the radio transmission methods used by Wi-Fi.|
|Bluetooth PHY|Defines radio transmission for Bluetooth devices.|

## Quick revision: numbers you should remember

|Protocol|Number|What the number means|
|---|---|---|
|TCP|6|IP protocol number|
|UDP|17|IP protocol number|
|ICMP|1|IPv4 protocol number|
|OSPF|89|IP protocol number|
|EIGRP|88|IP protocol number|
|GRE|47|IP protocol number|
|DNS|53|TCP/UDP port number|
|HTTP|80|TCP port number|
|HTTPS|443|TCP port number|
|SSH|22|TCP port number|
|BGP|179|TCP port number|
|RIP|520|UDP port number|

The three concepts to keep separate are:

- IP protocol number: Identifies the protocol carried in an IP packet.
    
- TCP/UDP port number: Identifies the application or service using TCP or UDP.
    
- EtherType: Identifies the payload protocol inside an Ethernet frame.
    

For official reference, see the IANA registries: [Protocol Numbers](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml), [Service Name and Transport Protocol Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml), and [IEEE EtherType values](https://standards-oui.ieee.org/ethertype/eth.txt).
