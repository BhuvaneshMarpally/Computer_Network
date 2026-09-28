## What is RIP?

**RIP (Routing Information Protocol)** is a **dynamic routing protocol** used by routers to automatically learn and exchange routes to different networks.

It is one of the older and simpler **Interior Gateway Protocols (IGPs)**.

*Instead of manually configuring every route, routers running RIP **share their routing information with neighboring routers**.*

> **RIP is a dynamic, interior gateway, distance-vector routing protocol.**

## RIP is a distance-vector protocol

The name describes the information used to choose a route:

- **Distance:** How far away is the destination? RIP measures this using **hop count**.
- **Vector:** In which direction should the packet go? This is the **next-hop router**.


![[../../../../Images/Pasted image 20260927212740.png]]

In every router the RIP  running at the port number 520, So whenever the other router want to send the routing table, it uses the UDP protocol and the port number 520.

Whenever the packet received to the router first it will see that the port number, then the OS of the Router think like this ==*"I need to send this packet to 520 port so, do i enabled the 520 port is enabled or not. if enable i send to that port otherwise i will drop this packet"*==

## Purpose of RIP
---
The main purpose of RIP is to allow routers to:
* Discover remote networks
* Exchange routing information
* Automatically learn routes
* Select a route to a destination
* Update their routing tables when routes change

## RIP Versions
---
```text
Dynamic Routing Protocol
        |
        |---- RIPv1
        |
        |---- RIPv2
```
			RIPv1
			
			* Classful routing protocol
			* Does **not** send subnet mask information in routing updates
			* Does not support VLSM
			* Uses broadcast for updates
			
			RIPv2
			
			* Classless routing protocol
			* Sends subnet mask information
			* Supports VLSM and CIDR
			* Uses multicast address **224.0.0.9** for RIP updates


## RIP Metric – Hop Count
---
RIP uses **hop count** as its routing metric.
A **hop** generally means passing through one router.

Example:
```text
Network A---R1---R2---R3---Network B
```

From R1 to Network B:
```text
R1 → R2 → R3 → Network B
```

The route has **2 router hops** before reaching the destination network.
==RIP generally prefers the route with the **lowest hop count**.==
### Maximum Hop Count
---
RIP has a maximum usable hop count of **15**.
```text
1–15 hops  → Reachable
16 hops    → Unreachable
```

This is an important limitation of RIP.

Because of this limitation, ==RIP is generally more suitable for **small networks** rather than large enterprise networks.==

## How RIP Works – Basic Idea
---
Suppose we have:
```text
LAN 1        LAN 2        LAN 3
  |            |            |
 R1 ---------- R2 ---------- R3
```

Initially, each router knows its directly connected networks.
Then RIP allows the routers to exchange routing information.
## What does RIP send?
---
**RIP sends route advertisements—not a literal copy of the entire `show ip route` output.**

Advertisements contain *destination information and a metric*. 
RIPv2 also includes a subnet mask and additional fields.

|Information|RIPv1|RIPv2|
|---|---|---|
|Destination network address|Yes|Yes|
|Metric|Yes|Yes|
|Subnet mask|No|Yes|
|Next-hop field|No|Yes|
|Route tag|No|Yes|
RIP normally sends its full set of **eligible advertised routes** every **30 seconds**.
## RIP Routing Updates
---
RIP periodically sends routing information to its neighbors.
By default, RIP sends updates approximately every **30 seconds**.
The updates contain information about the routes the router knows.
## RIP Routing Table
---
After learning routes through RIP, the router places the learned routes into its routing table.

You can check the routing table using:
```text
Router# show ip route
```

Routes learned through RIP are normally identified by:
```text
R
```

Example:
```text
R    192.168.3.0/24 [120/2] via 192.168.2.2
```

Here:
* **R** → route learned through RIP
* **120** → RIP administrative distance
* **2** → RIP metric (hop count)
* **via 192.168.2.2** → next-hop router
## Automatic summarization
---
Automatic summarization advertises a classful major network when subnet routes cross that major network's boundary.

For example, these subnets:
```
172.16.1.0/24
172.16.2.0/24
```

Can be advertised outside the `172.16.0.0/16` major network as:
```
172.16.0.0/16
```

That summary hides the individual `/24` routes.

In Cisco RIPv2 labs, use:
```
Router(config-router)# no auto-summary
```

This allows the actual subnet routes and masks to be advertised across classful boundaries.

==**`version 2` enables RIPv2. `no auto-summary` separately disables automatic classful summarization==.**

## RIP loop-prevention mechanisms
---

|Mechanism|Purpose|
|---|---|
|Split horizon|Does not advertise a learned route back out the interface where it was learned|
|Route poisoning|Advertises a failed route with metric 16|
|Poison reverse|Advertises a route back toward its source with metric 16|
|Triggered updates|Sends changed routing information without waiting for the next periodic update|
|Hold-down timer|Temporarily restricts acceptance of potentially misleading updates about a failed route|

These mechanisms help reduce routing loops and incorrect information during convergence.

**Convergence** means the routers have updated their routing information to reflect the network's current state.

## Common Cisco RIP timers
---

|Timer|Default|Purpose|
|---|---|---|
|Update|30 seconds|Sends periodic routing updates|
|Invalid|180 seconds|Marks a route invalid if no fresh update arrives|
|Hold-down|180 seconds|Helps prevent unstable information from reinstating a failed route|
|Flush|240 seconds|Removes a stale route from the routing table|

The invalid and flush timers are measured from the last valid update. They are not added together.
#  Important RIP Characteristics
---

| Feature                 | RIP                          |
| ----------------------- | ---------------------------- |
| Full form               | Routing Information Protocol |
| Type                    | Dynamic routing protocol     |
| Classification          | IGP                          |
| Routing type            | Distance vector              |
| Metric                  | Hop count                    |
| Maximum usable hops     | 15                           |
| 16 hops                 | Unreachable                  |
| Default update interval | 30 seconds                   |
| Administrative distance | 120                          |
| Protocol                | UDP                          |
| Port                    | 520                          |
| RIPv2 multicast         | 224.0.0.9                    |
| RIPv1                   | Classful                     |
| RIPv2                   | Classless                    |

