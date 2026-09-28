# RIP — Routing Information Protocol

## 1. What is RIP?
![](../../../Pasted%20image%2020260928131755.png)

**RIP (Routing Information Protocol)** is a dynamic routing protocol that allows routers to automatically learn and exchange routes to remote networks.

Instead of manually configuring every route, routers share routing information with their neighbors.

> **RIP is a dynamic, interior gateway, distance-vector routing protocol.**

| Classification                  | Meaning                                                         |
| ------------------------------- | --------------------------------------------------------------- |
| Dynamic                         | Learns and updates routes automatically                         |
| Interior Gateway Protocol — IGP | Used within an organization’s routing domain                    |
| Distance vector                 | Learns destinations, their distances and the next-hop direction |

RIPv1 and RIPv2 work with **IPv4**. A separate version called **RIPng** supports IPv6.

## 2. Purpose of RIP

RIP helps routers:

* Discover remote networks.
* Exchange reachability information.
* Select paths based on hop count.
* Update routes when the network changes.
* Remove routes that are no longer reachable.

**RIP learns routes. The router uses its routing table to forward ordinary traffic.** User packets are not encapsulated inside RIP updates.

## 3. Distance vector and hop count

### Distance

The distance is the **hop count**: how many routers must be crossed to reach the destination network.

### Vector

The vector is the **next-hop direction**: which neighboring router should receive the packet.

Example path:

**R1 → R2 → R3 → Network B**

| Router | Route to Network B  |              Hop count |
| ------ | ------------------- | ---------------------: |
| R3     | Directly connected  | No intermediate router |
| R2     | Through R3          |                      1 |
| R1     | Through R2, then R3 |                      2 |

R1 knows that Network B is two hops away through R2. It does **not** learn a complete topology map or a list of every router in the path.

### Route selection

RIP prefers the route with the **lowest hop count**.

| Available path | Hop count | RIP preference |
| -------------- | --------: | -------------- |
| Path A         |         2 | Preferred      |
| Path B         |         4 | Not preferred  |

RIP does not consider link bandwidth in its metric. A slow two-hop path can be preferred over a fast three-hop path.

### Maximum hop count

| Metric | Meaning     |
| ------ | ----------- |
| 1–15   | Reachable   |
| 16     | Unreachable |

This limit makes RIP unsuitable for large networks.

### Equal-cost load balancing

If multiple paths to the same destination have the same hop count, Cisco routers can install multiple next hops and share traffic between them, subject to the configured maximum-path limit.

## 4. How RIP learns routes

Assume Network B is directly connected to R3.

1. R3 already knows Network B from its active interface configuration.
2. R3 advertises Network B to R2.
3. R2 learns a one-hop route through R3.
4. R2 advertises its reachability to R1.
5. R1 learns a two-hop route through R2.
6. Periodic updates refresh these routes.
7. If the network fails, updates and timers help remove the unavailable route.

**Directly connected routes remain connected routes in the routing table.** Enabling RIP does not change their route code from `C` to `R`.

### Does RIP establish a neighbor adjacency?

RIP does not establish an OSPF-style adjacency or perform a TCP handshake before sending normal updates.

A router can send an update even if no other router on the segment is running RIP.

For successful route learning, the receiver must accept the update’s version and pass any other applicable checks.

## 5. What RIP sends

**RIP sends route advertisements, not a literal copy of `show ip route`.**

| Route-entry information | RIPv1 | RIPv2 |
| ----------------------- | ----- | ----- |
| Destination address     | Yes   | Yes   |
| Metric                  | Yes   | Yes   |
| Subnet mask             | No    | Yes   |
| Next-hop field          | No    | Yes   |
| Route tag               | No    | Yes   |

* **Next-hop field:** Identifies a next hop to use. In RIPv2, `0.0.0.0` means use the router that sent the update.
* **Route tag:** Carries a label that can distinguish routes, such as routes imported from another routing protocol.

RIP periodically advertises its full set of **eligible routes**, approximately every **30 seconds**.

Eligible routes depend on configuration:

* Split horizon can suppress advertisements toward the source of a route.
* Filtering can exclude routes.
* Summarization can combine routes.
* Routes from another routing protocol generally require redistribution.

RIP uses **Request** messages to ask for routing information and **Response** messages to provide it. Periodic updates are Response messages.

## 6. UDP port 520 and packet processing

**RIPv1 and RIPv2 use UDP port `520`.**

A router running RIP has a RIP process listening on this UDP port. RIP is not automatically running on every router.

A port is a software identifier, not a physical interface.

### How a received update reaches RIP

The following is a simplified receiving sequence:

1. **Ethernet processing:** Check the frame and its destination MAC.
2. **IP processing:** Check whether the destination is accepted for local reception.
3. **Transport identification:** The IP protocol field identifies UDP.
4. **UDP processing:** Examine destination port `520`.
5. **Application delivery:** Deliver the payload to the RIP process.
6. **RIP processing:** Validate the update and evaluate its routes.

If no application is listening on UDP port `520`, the packet is not delivered to a RIP process and is discarded.

> **The router does not check the UDP port first. Ethernet and IP processing come before UDP delivery.**

Receiving an update does not guarantee that its routes will be installed. Version checks, authentication, filtering and route preference can affect the result.

## 7. RIPv1

**RIPv1 is a classful distance-vector routing protocol.**

Its main limitation is:

> **RIPv1 does not include subnet masks in route advertisements.**

The receiving router must infer the mask.

### How RIPv1 infers a mask

For ordinary subnet routes:

| Advertised destination                                                | Mask interpretation                        |
| --------------------------------------------------------------------- | ------------------------------------------ |
| Belongs to the same classful major network as the receiving interface | Uses the receiving interface’s subnet mask |
| Belongs to a different classful major network                         | Uses the natural classful mask             |

This describes ordinary subnet advertisements; default routes and host routes involve additional handling.

### Natural classful masks

| Class | First octet | Natural mask            |
| ----- | ----------- | ----------------------- |
| A     | 1–126       | `/8` — `255.0.0.0`      |
| B     | 128–191     | `/16` — `255.255.0.0`   |
| C     | 192–223     | `/24` — `255.255.255.0` |

### Example: `/26` subnetting

R1’s receiving interface:

```text
IP address: 192.168.1.1
Subnet mask: 255.255.255.192 (/26)
```

Received RIPv1 route information:

```text
Destination: 192.168.1.128
Subnet mask: Not included
```

Both belong to the classful major network:

```text
192.168.1.0/24
```

R1 therefore uses its receiving interface’s `/26` mask and interprets the destination as:

```text
192.168.1.128/26
```

**R1 inferred `/26` from its own configuration. Its neighbor did not send that mask.**

If the actual remote subnet uses a different mask, the inference can be wrong.

### RIPv1 limitations

* Does not support VLSM.
* Does not support classless route advertisements.
* Summarizes at classful boundaries.
* Cannot reliably represent discontiguous subnets separated by another major network.
* Does not support authentication.
* Normally sends updates using broadcast `255.255.255.255`.

**Classful does not mean that interfaces can use only `/8`, `/16` or `/24`.** Consistent fixed-length subnetting, such as `/26`, can work within a major network.

## 8. RIPv2

**RIPv2 is a classless distance-vector routing protocol.**

Its main improvement is:

> **RIPv2 includes the subnet mask in each route advertisement.**

Example:

```text
Destination: 192.168.1.128
Subnet mask: 255.255.255.192
Next hop: 0.0.0.0
```

The receiver directly interprets the destination as:

```text
192.168.1.128/26
```

It does not need to infer the mask.

RIPv2 supports:

* **VLSM:** Different subnet sizes within the same major network.
* **CIDR:** Classless prefixes and route aggregation.
* **Authentication:** Checking routing updates using configured authentication.
* **Discontiguous networks:** When summarization and routing are configured appropriately.
* **Multicast updates:** Normally sent to `224.0.0.9`.

`224.0.0.9` is reserved for **RIPv2 routers**. ([iana.org][1])

Authentication does not encrypt ordinary user traffic. RIPv2 can also use broadcast or unicast updates in particular configurations.

## 9. Automatic summarization

Automatic summarization combines subnet information into the **classful major network** when advertising across that major network’s boundary.

Example subnets:

```text
172.16.1.0/24
172.16.2.0/24
```

When advertised through an interface in a different major network, they can be summarized as:

```text
172.16.0.0/16
```

The receiver then sees the summary instead of the individual `/24` routes.

### Why this can cause problems

A **discontiguous network** has subnets of the same major network separated by a different major network.

For example, separate sites use `172.16.1.0/24` and `172.16.2.0/24`, but connect through a `10.0.0.0/8` network. Advertising only `172.16.0.0/16` can hide which site contains each subnet.

### Disable automatic summarization in RIPv2

```text
Router(config-router)# no auto-summary
```

This allows subnet prefixes to be advertised across classful boundaries without automatic classful summarization. ([Cisco Systems][2])

**`version 2` selects RIPv2. `no auto-summary` is a separate setting.**

Disabling automatic summarization does not prevent explicitly configured manual summaries.

## 10. RIPv1 broadcast and RIPv2 multicast

| Address field            | RIPv1 normal update | RIPv2 normal update |
| ------------------------ | ------------------- | ------------------- |
| Destination IP           | `255.255.255.255`   | `224.0.0.9`         |
| Destination Ethernet MAC | `FF:FF:FF:FF:FF:FF` | `01:00:5E:00:00:09` |
| Destination UDP port     | `520`               | `520`               |

**Broadcast:** Addresses everyone on the local broadcast domain.

**Multicast:** Addresses a particular group of receivers.

RIPv1’s broadcast choice and its classful behavior are separate characteristics. Missing subnet masks do not inherently require broadcasting.

### Example shared LAN

R1, R2 and two PCs connect to one switch in the same VLAN:

| Device | Interface address | RIPv2 enabled? |
| ------ | ----------------- | -------------- |
| R1     | `192.168.1.1/24`  | Yes            |
| R2     | `192.168.1.2/24`  | Yes            |
| PC1    | `192.168.1.10/24` | No             |
| PC2    | `192.168.1.11/24` | No             |

### Step 1: R2 listens to the group

R2’s RIPv2 software registers interest in `224.0.0.9` on the interface.

R2 can therefore accept both:

* `192.168.1.2` — its own interface address.
* `224.0.0.9` — the RIPv2 multicast group address.

The group address does not replace the interface address or become a second configured unicast IP.

### Step 2: R1 sends the update

R1 sends a packet addressed to `224.0.0.9`.

It does not need to know every group member individually.

On Ethernet, the multicast IP maps to `01:00:5E:00:00:09`. **ARP is not needed to resolve this multicast destination.**

### Step 3: The switch forwards it

For this special local multicast range, the switch normally forwards the frame to the other forwarding ports in the same VLAN, including the PC ports.

Therefore, **RIPv2 multicast can produce the same switch-port copies as broadcast in this example**. This is different from multicast groups for which IGMP snooping restricts forwarding to interested ports. ([RFC Editor][3])

### Step 4: Receivers filter it

| Receiver    | Processing                                                        |
| ----------- | ----------------------------------------------------------------- |
| R2          | Accepts the group destination and delivers valid updates to RIP   |
| PC1 and PC2 | Can reject the unwanted multicast at the network card or IP layer |

Hardware filtering varies. If the network card passes the frame upward, the IP stack can reject it because the PC has not joined the group.

### Why multicast helps

For a PC that is not running RIP, assuming no earlier firewall drop:

| Check           | RIPv1 broadcast                     | RIPv2 multicast                    |
| --------------- | ----------------------------------- | ---------------------------------- |
| Destination MAC | Broadcast is accepted               | Unwanted multicast can be filtered |
| Destination IP  | Broadcast includes the PC           | PC is not a member of the group    |
| UDP processing  | No listener on port `520` → discard | May already have been discarded    |

**Both frames can reach the PC’s cable. Multicast allows rejection based on group membership before delivery to UDP.**

These RIPv2 multicast updates stay on the local link. A receiving router creates its own updates on other interfaces; it does not simply forward the original multicast packet across the network.

## 11. Passive interfaces

A passive RIP interface stops normal outgoing RIP updates on that interface.

```text
Router(config)# router rip
Router(config-router)# passive-interface gigabitEthernet0/0
```

The connected LAN can still be advertised through other RIP interfaces.

### When to use it

Use it on a LAN interface that connects only to end devices and has no router requiring RIP updates.

| Feature           | Effect                                                |
| ----------------- | ----------------------------------------------------- |
| Multicast         | Sends updates addressed to a group                    |
| Passive interface | Stops normal RIP update transmission on the interface |

If R2 and PCs share R1’s LAN interface, making that interface passive also stops normal updates from R1 to R2.

**In Cisco IOS RIP, passive does not normally block receiving RIP updates.** It is not a substitute for an inbound filter.

## 12. Loop prevention and convergence

**Convergence** is the process of routers updating their routing information after a network change until their routes reflect the new state.

Distance-vector protocols can suffer from **counting to infinity**: routers repeatedly advertise increasingly worse metrics because they mistakenly believe a failed network is reachable through each other.

RIP treats metric `16` as infinity.

| Mechanism         | Purpose                                                                            |
| ----------------- | ---------------------------------------------------------------------------------- |
| Split horizon     | Suppresses a learned route when advertising out the interface where it was learned |
| Route poisoning   | Advertises an unavailable route with metric `16`                                   |
| Poison reverse    | Advertises a route back toward its source with metric `16`                         |
| Triggered updates | Sends changes without waiting for the next periodic update                         |
| Hold-down timer   | Restricts acceptance of potentially misleading information about a failed route    |

Triggered updates can be briefly delayed or rate-limited. They are not guaranteed to be instantaneous.

These mechanisms reduce routing problems but do not eliminate every possible temporary loop.

## 13. Common Cisco IOS RIP timers

| Timer     |     Default | Purpose                                                      |
| --------- | ----------: | ------------------------------------------------------------ |
| Update    |  30 seconds | Sends periodic updates                                       |
| Invalid   | 180 seconds | Marks a route invalid if updates stop                        |
| Hold-down | 180 seconds | Helps prevent unstable route information from being accepted |
| Flush     | 240 seconds | Removes the stale route                                      |

These are common **Cisco IOS defaults**; timer behavior can differ across implementations. ([cisco.com][4])

The invalid and flush timers are measured from the last valid update. They are **not added together**.

A known failure can trigger action earlier than the invalid timer.

## 14. Reading the routing table

```text
R    192.168.2.0/24 [120/1] via 10.0.0.2, 00:00:12, GigabitEthernet0/1
```

| Part                 | Meaning                            |
| -------------------- | ---------------------------------- |
| `R`                  | Learned through RIP                |
| `192.168.2.0/24`     | Destination prefix                 |
| `120`                | Cisco RIP administrative distance  |
| `1`                  | Hop-count metric                   |
| `10.0.0.2`           | Next-hop router                    |
| `00:00:12`           | Time since the route’s last update |
| `GigabitEthernet0/1` | Outgoing interface                 |

### Administrative distance versus metric

* **Administrative distance:** Compares route sources for the same prefix.
* **RIP metric:** Compares RIP paths to that prefix.

Cisco normally prefers OSPF with AD `110` over RIP with AD `120` for the **same prefix**, even if RIP has fewer hops.

When forwarding a packet, the router uses the **longest matching prefix** among installed routes. It does not simply choose the lowest AD across all matching prefix lengths.

## 15. RIPv1 versus RIPv2

| Feature                  | RIPv1                       | RIPv2                    |
| ------------------------ | --------------------------- | ------------------------ |
| Routing type             | Distance vector             | Distance vector          |
| Classful/classless       | Classful                    | Classless                |
| Metric                   | Hop count                   | Hop count                |
| Maximum reachable metric | 15                          | 15                       |
| Unreachable metric       | 16                          | 16                       |
| Mask in updates          | No                          | Yes                      |
| VLSM and CIDR            | No                          | Yes                      |
| Normal destination       | Broadcast `255.255.255.255` | Multicast `224.0.0.9`    |
| Authentication           | No                          | Supported                |
| Transport                | UDP                         | UDP                      |
| Port                     | 520                         | 520                      |
| Periodic update interval | Approximately 30 seconds    | Approximately 30 seconds |
| Cisco default AD         | 120                         | 120                      |

## 16. Cisco Packet Tracer configuration

Assume the interfaces already have IP addresses and are up:

| Router interface | Connected network | Purpose               |
| ---------------- | ----------------- | --------------------- |
| G0/0             | `192.168.1.0/24`  | PC LAN                |
| G0/1             | `10.0.0.0/30`     | Router-to-router link |

Choose one of the following version configurations.

### Enter global configuration mode

```text
Router> enable
Router# configure terminal
Router(config)#
```

### RIPv1 configuration

```text
Router(config)# router rip
Router(config-router)# version 1
Router(config-router)# network 192.168.1.0
Router(config-router)# network 10.0.0.0
Router(config-router)# passive-interface gigabitEthernet0/0
Router(config-router)# end
```

### RIPv2 configuration

```text
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# no auto-summary
Router(config-router)# network 192.168.1.0
Router(config-router)# network 10.0.0.0
Router(config-router)# passive-interface gigabitEthernet0/0
Router(config-router)# end
```

### What the commands mean

| Command               | Purpose                                             |
| --------------------- | --------------------------------------------------- |
| `router rip`          | Starts/configures the RIP routing process           |
| `version 1`           | Selects RIPv1                                       |
| `version 2`           | Selects RIPv2                                       |
| `no auto-summary`     | Disables automatic classful summarization           |
| `network …`           | Selects local interfaces for RIP by major network   |
| `passive-interface …` | Stops normal outgoing RIP updates on that interface |
| `end`                 | Returns to privileged EXEC mode                     |

### Understanding the `network` command

```text
network 10.0.0.0
```

Matches local interfaces whose addresses belong to the classful major network `10.0.0.0/8`, even when those interfaces use `/30` or another subnet mask.

It enables RIP operation on matching interfaces and makes their connected networks eligible for advertisement.

**Enter local major networks, not remote destinations you want to learn.** RIP’s `network` command does not use a wildcard mask.

### Optional: per-interface version control

Cisco IOS can override the global send and receive versions for an interface:

```text
Router(config)# interface gigabitEthernet0/1
Router(config-if)# ip rip send version 2
Router(config-if)# ip rip receive version 2
Router(config-if)# end
```

Sending and receiving are separate settings. A version mismatch does not break the physical link, but it can prevent route learning. ([Cisco][5])

### Save the configuration

```text
Router# copy running-config startup-config
```

## 17. Verification commands

Run these in privileged EXEC mode:

| Command                   | Purpose                                               |
| ------------------------- | ----------------------------------------------------- |
| `show ip route`           | Displays installed routes                             |
| `show ip route rip`       | Displays installed RIP routes                         |
| `show ip protocols`       | Checks versions, networks, timers and routing sources |
| `show ip interface brief` | Checks IP addresses and interface status              |
| `show running-config`     | Reviews configuration                                 |
| `ping 10.0.0.2`           | Tests neighbor reachability                           |
| `show ip rip database`    | Examines RIP’s routing database, where supported      |
| `traceroute 192.168.2.1`  | Examines the forwarding path                          |

### Observe RIP updates

```text
Router# debug ip rip
```

Stop debugging after checking:

```text
Router# undebug all
```

The RIP database and the main routing table are not identical: a route known to RIP might lose to a preferred route source and therefore not appear as an installed `R` route.

## 18. Troubleshooting checklist

If RIP routes are missing, check:

1. **Interfaces:** Are the relevant interfaces `up/up`?
2. **Addressing:** Are directly connected neighbors correctly addressed in the same subnet?
3. **Network commands:** Do they match the intended local interfaces?
4. **Versions:** Does each router accept the version its neighbor sends?
5. **Passive interfaces:** Was the router-to-router interface made passive accidentally?
6. **Summarization:** Is automatic summarization hiding the required subnet?
7. **Filters and authentication:** Are updates being blocked or rejected?
8. **Hop count:** Has the route reached metric `16`?
9. **Other route sources:** Is a connected, static or other preferred route installed?
10. **Return path:** Does the remote side have a route back?

A successful ping to the immediate neighbor verifies basic connectivity; it does not prove that RIP is configured correctly.

## 19. Quick revision

* **RIP:** Dynamic, IGP, distance vector.
* **Metric:** Hop count.
* **Maximum:** 15 reachable; 16 unreachable.
* **Transport:** UDP `520`.
* **Cisco AD:** `120`.
* **Updates:** Approximately every 30 seconds.
* **RIPv1:** No mask in updates; normally broadcasts.
* **RIPv2:** Includes masks; normally multicasts to `224.0.0.9`.
* **Passive interface:** Stops normal outgoing updates while its LAN can still be advertised elsewhere.
* **`no auto-summary`:** Disables automatic classful summarization.
* **RIP learns routes; the routing table guides packet forwarding.**

[1]: https://www.iana.org/assignments/multicast-addresses?utm_source=chatgpt.com "IPv4 Multicast Address Space"
[2]: https://www.cisco.com/en/US/docs/ios/zz_trash/config_modules/irr_cfg_info_prot_xe.html?utm_source=chatgpt.com "Configuring Routing Information Protocol  [Networking Software (IOS & NX-OS)]"
[3]: https://www.rfc-editor.org/rfc/rfc4541?utm_source=chatgpt.com "RFC 4541: Considerations for Internet Group Management Protocol (IGMP) and Multicast Listener Discovery (MLD) Snooping Switches"
[4]: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_rip/configuration/xe-16-12/irr-xe-16-12-book.pdf?utm_source=chatgpt.com "IP Routing: RIP Configuration Guide, Cisco IOS XE Gibraltar 16.12.x"
[5]: https://team-development.cisco.com/c/en/us/td/docs/switches/lan/catalyst9600/software/release/17-12/configuration_guide/rtng/b_1712_rtng_9600_cg/configuring_rip.html?utm_source=chatgpt.com "IP Routing Configuration Guide, Cisco IOS XE Dublin 17.12.x (Catalyst 9600 Switches) - Configuring RIP [Cisco Catalyst 9600 Series Switches]"
