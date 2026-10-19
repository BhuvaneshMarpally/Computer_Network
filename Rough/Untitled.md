# ICMP — Internet Control Message Protocol

ICMP (Internet Control Message Protocol) is a *network-layer* control *protocol* used by hosts and routers to report network errors and exchange diagnostic information.

`“It works alongside IP and is commonly used by tools such as `*ping*` and `*traceroute*` to troubleshoot network connectivity.”`

```
“IP is responsible for addressing and forwarding packets, while ICMP helps communicate network errors and diagnostic information. Tools such as ping use ICMP Echo Request and Echo Reply messages to test reachability, while traceroute uses TTL expiration and ICMP responses to discover the network path.”
```


>- IPv4 protocol number: 1
>- IPv6 protocol number: 58 (ICMPv6)
>- Transport protocol: Neither TCP nor UDP
>- Common utility: `ping`
>- Other common utility: `traceroute` / `tracepath`

ICMP does not transport application data like TCP or UDP. Its primary purpose is network control, error reporting, and diagnostics.

## 2. Why do we need ICMP?

IP provides best-effort delivery. A packet can be lost or discarded without IP itself providing a detailed explanation to the sender.

ICMP helps report certain conditions, such as:


- Destination network or host unreachable.
- Port unreachable (for applicable UDP traffic).
- Time-to-live (TTL) expired.
- Invalid IP header parameters.
- Successful reachability checks using Echo Request and Echo Reply.

Important: ICMP reports many network conditions, but it does not guarantee delivery, repair connectivity, or report every packet loss.

## 3. ICMP packet structure and field sizes

An ICMP message is carried inside an IP packet. For IPv4, the IP header identifies ICMP using protocol number `1`.

Echo Request / Echo Reply format

![[Pasted image 20261019195121.png|670]]

| Field           | Size              | Function                                  |
| --------------- | ----------------- | ----------------------------------------- |
| Type            | 8 bits (1 byte)   | Identifies the ICMP message               |
| Code            | 8 bits (1 byte)   | Specifies the message subtype or reason   |
| Checksum        | 16 bits (2 bytes) | Checks the integrity of the ICMP message  |
| Identifier      | 16 bits (2 bytes) | Helps associate requests and replies      |
| Sequence Number | 16 bits (2 bytes) | Helps distinguish individual requests     |
| Data / Payload  | Variable          | Carries the bytes included in the message |

The Echo header is 8 bytes (64 bits), excluding the payload. Not every ICMP type uses the Identifier and Sequence Number; the remaining header format depends on the message type.

### Important ICMP types

| Type | Message                 | Purpose                                                                     |
| ---- | ----------------------- | --------------------------------------------------------------------------- |
| 0    | Echo Reply              | Responds to an Echo Request                                                 |
| 3    | Destination Unreachable | Reports that a destination or delivery path is unreachable                  |
| 5    | Redirect                | Informs a host of a potentially better gateway                              |
| 8    | Echo Request            | Requests an Echo Reply                                                      |
| 11   | Time Exceeded           | Reports that the IP TTL expired in transit or fragment reassembly timed out |
| 12   | Parameter Problem       | Reports a problem with an IP header parameter                               |

These are common IPv4 ICMP message types.

## 4. Complete ICMP workflow

Let's use a practical example.

- PC1: `192.168.10.10/24`
- PC2: `192.168.10.20/24`
- Both PCs belong to the same VLAN and subnet.
- PC1 runs `ping 192.168.10.20`.

![[Pasted image 20261019195730.png]]

The normal Echo Reply behavior, including returning the data from the request, is specified in RFC 792 and RFC 1122.
### Workflow in short

`ping command` → ICMP Echo Request → IPv4 packet → network forwarding → destination processes request → ICMP Echo Reply → source measures round-trip time.

Remember: ICMP does not establish a connection like TCP. Each Echo Request is an individual message, and the destination can reply if it receives and permits the request.

## 5. Where does the ICMP payload come from?

The `ping` program typically generates the payload bytes. The bytes may contain a recognizable pattern or sequence; they do not have to be meaningful text.

For example, on many Linux systems:

```
ping -c 1 -s 56 192.168.10.20
```

Here, `-s 56` requests a payload of 56 bytes.

| Component                     | Size     |
| ----------------------------- | -------- |
| IPv4 header (without options) | 20 bytes |
| ICMP Echo header              | 8 bytes  |
| ICMP payload                  | 56 bytes |
| Total IPv4 packet             | 84 bytes |

%%This assumes no IPv4 options or fragmentation and excludes the Ethernet header and other link-layer overhead.

The receiver normally copies the Echo Request's data into the Echo Reply. The payload helps verify that data sent to the destination comes back intact.%%
## 6. ICMP error-message workflow

ICMP is not limited to `ping`. Routers and hosts can also generate ICMP error messages when certain problems occur.

Example: PC1 sends an IP packet to a remote network, but a router cannot forward it.

![[Pasted image 20261019200456.png|641]]

The router does not always send an error. Some conditions cause packets to be silently discarded, and filtering can block ICMP messages.
## 7. Practical ICMP troubleshooting

Use these commands on Ubuntu or another Linux system.

| Command                    | Purpose                                                    |
| -------------------------- | ---------------------------------------------------------- |
| `ping -c 4 192.168.10.20`  | Send four Echo Requests                                    |
| `ip addr`                  | Check the local interface addresses                        |
| `ip route`                 | Check the routing table                                    |
| `ip neigh`                 | Inspect IPv4/IPv6 neighbor entries                         |
| `traceroute 192.168.10.20` | Investigate the path toward a destination, where supported |

For packet-level inspection, you can use Wireshark with the display filter:

```
icmp
```

Or capture ICMP traffic in a Linux terminal:

```
sudo tcpdump -ni any icmp
```

~~~When a ping fails, check the local IP and subnet mask, the route and default gateway, VLAN configuration, destination availability, and firewall rules. A failed ping alone does not prove that the destination is offline; ICMP may be filtered even when other services work.
~~~
## 8. Quick revision notes

- ICMP means Internet Control Message Protocol.
- IPv4 identifies ICMP with protocol number `1`.
- ICMP works alongside IP and does not use TCP or UDP ports.
- Type identifies the message; Code provides additional detail.
- The common ICMP header is 4 bytes.
- The Echo Request/Reply header is 8 bytes, including Identifier and Sequence Number.
- The payload is variable-length data generated by the sending program.
- Echo Request = Type `8`, Code `0`.
- Echo Reply = Type `0`, Code `0`.
- Destination Unreachable = Type `3`.
- Time Exceeded = Type `11`.
- The Echo Reply normally includes the same payload as the request.
- ICMP helps with diagnostics and error reporting, but does not guarantee delivery.

Topic progress: ICMP introduction, packet fields and sizes, payload, request/reply workflow, error reporting, and basic troubleshooting are covered.

Next topic: ICMP in depth — `ping` packet analysis in Wireshark, TTL behavior, and how `traceroute` uses ICMP Time Exceeded messages.