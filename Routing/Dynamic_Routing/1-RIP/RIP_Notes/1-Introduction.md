# RIP (Routing Information Protocol) – Introduction

## 1. What is RIP?

**RIP (Routing Information Protocol)** is a **dynamic routing protocol** used by routers to automatically learn and exchange routes to different networks.

Instead of manually configuring every route, routers running RIP **share their routing information with neighboring routers**.

### Simple Example

```text
PC Network A
     |
    R1
     |
    R2
     |
PC Network B
```

Without dynamic routing, we would manually tell **R1** how to reach Network B and **R2** how to reach Network A.

With RIP:

```text
R1  <------ RIP ------>  R2
 |                       |
Learns routes         Learns routes
automatically         automatically
```

---

## 2. Full Form

**RIP = Routing Information Protocol**

It is one of the older and simpler **Interior Gateway Protocols (IGPs)**.

---

## 3. Purpose of RIP

The main purpose of RIP is to allow routers to:

* Discover remote networks
* Exchange routing information
* Automatically learn routes
* Select a route to a destination
* Update their routing tables when routes change

---

## 4. Type of Routing Protocol

RIP is classified as a:

```text
Dynamic Routing Protocol
        |
        |---- Interior Gateway Protocol (IGP)
                    |
                    |---- Distance-Vector Routing Protocol
```

So remember:

> **RIP is a dynamic, interior gateway, distance-vector routing protocol.**

---

## 5. RIP Metric – Hop Count

RIP uses **hop count** as its routing metric.

A **hop** generally means passing through one router.

Example:

```text
Network A
   |
   R1
   |
   R2
   |
   R3
   |
Network B
```

From R1 to Network B:

```text
R1 → R2 → R3 → Network B
```

The route has **2 router hops** before reaching the destination network.

RIP generally prefers the route with the **lowest hop count**.

---

## 6. Maximum Hop Count

RIP has a maximum usable hop count of **15**.

```text
1–15 hops  → Reachable
16 hops    → Unreachable
```

This is an important limitation of RIP.

Because of this limitation, RIP is generally more suitable for **small networks** rather than large enterprise networks.

---

## 7. How RIP Works – Basic Idea

Suppose we have:

```text
LAN 1        LAN 2        LAN 3
  |            |            |
 R1 ---------- R2 ---------- R3
```

Initially, each router knows its directly connected networks.

Then RIP allows the routers to exchange routing information.

For example:

```text
R1 → R2:
"I know how to reach LAN 1."

R2 → R3:
"I know how to reach LAN 1 and LAN 2."

R3 → R2:
"I know how to reach LAN 3."
```

The routers gradually learn about remote networks and add appropriate routes to their routing tables.

---

## 8. RIP Versions

There are two commonly discussed versions:

### RIPv1

* Classful routing protocol
* Does **not** send subnet mask information in routing updates
* Does not support VLSM
* Uses broadcast for updates

### RIPv2

* Classless routing protocol
* Sends subnet mask information
* Supports VLSM and CIDR
* Uses multicast address **224.0.0.9** for RIP updates

For modern learning and Packet Tracer labs, **RIPv2** is usually the more useful version to practice.

---

## 9. RIP Routing Updates

RIP periodically sends routing information to its neighbors.

By default, RIP sends updates approximately every **30 seconds**.

The updates contain information about the routes the router knows.

For example:

```text
R1
 |
 |---- Network 192.168.1.0/24
 |---- Network 192.168.2.0/24
 |---- Network 192.168.3.0/24
 |
 ↓
R2
```

R2 can use this information to learn routes that were previously unknown to it.

---

## 10. RIP Routing Table

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

---

# 11. Important RIP Characteristics

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
| RIPv2 multicast         | 224.0.0.9                    |
| RIPv1                   | Classful                     |
| RIPv2                   | Classless                    |

---

# 12. Simple Definition for Your Notes

> **RIP (Routing Information Protocol) is a dynamic distance-vector interior gateway routing protocol that allows routers to automatically exchange routing information and select routes based primarily on hop count.**

---

## 13. RIP Learning Flow

Remember RIP like this:

```text
              RIP
               |
       Dynamic Routing
               |
              IGP
               |
        Distance Vector
               |
          Hop Count
               |
     Lowest Hop Count
               |
       Selected Route
```

### Next topics to learn in order

For your RIP lab, I would learn it in this order:

**RIP basics → RIPv1 vs RIPv2 → RIP working → RIP timers → RIP routing table → RIPv2 configuration → `show ip route` → `show ip protocols` → troubleshooting → Packet Tracer RIP lab.**
