## Cisco Packet Tracer configuration

Assume a router has interfaces in:
```
192.168.1.0/24
10.0.0.0/30
```

### Enter configuration mode
```
Router> enable
Router# configure terminal
Router(config)#
```

### RIPv1
```
Router(config)# router rip
Router(config-router)# version 1
Router(config-router)# network 192.168.1.0
Router(config-router)# network 10.0.0.0
Router(config-router)# end
```

### RIPv2
```
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# no auto-summary
Router(config-router)# network 192.168.1.0
Router(config-router)# network 10.0.0.0
Router(config-router)# end
```

### What does the `network` command do?

It selects local interfaces belonging to the specified **classful major network** for RIP operation and makes their connected networks eligible for advertisement.

For example:
```
network 10.0.0.0
```

Matches local interfaces with addresses in `10.0.0.0/8`.

**You enter your router's local networks here. You do not enter every remote network you want to learn.**

RIP's `network` command does not take a wildcard mask.

### Stop sending updates toward a PC LAN

If `GigabitEthernet0/0` connects only to end devices:
```
Router(config)# router rip
Router(config-router)# passive-interface gigabitEthernet0/0
```

The router stops sending RIP updates on that interface. Its LAN network can still be advertised through other RIP interfaces.

### Save configuration
```
Router# copy running-config startup-config
```

## Verification commands

Run these in privileged EXEC mode:

|Command|What to check|
|---|---|
|`show ip route`|Connected and learned routes|
|`show ip route rip`|Routes learned through RIP|
|`show ip protocols`|RIP version, timers, networks and summarization|
|`show ip interface brief`|Interface addresses and up/down status|
|`show running-config`|Actual RIP configuration|
|`ping`|Whether the destination is reachable|

## 14. Reading a RIP route

Example:

```
R    192.168.2.0/24 [120/1] via 10.0.0.2, 00:00:12, GigabitEthernet0/1
```

|Part|Meaning|
|---|---|
|`R`|Route learned through RIP|
|`192.168.2.0/24`|Destination network and prefix length|
|`120`|Administrative distance|
|`1`|RIP metric: one hop|
|`10.0.0.2`|Next-hop router|
|`00:00:12`|Time since the last update for this route|
|`GigabitEthernet0/1`|Outgoing interface|

**Administrative distance compares route sources; hop count compares RIP paths.**

For example, Cisco normally prefers an OSPF route with AD 110 over a RIP route with AD 120 **for the same destination prefix**, even if RIP reports fewer hops.