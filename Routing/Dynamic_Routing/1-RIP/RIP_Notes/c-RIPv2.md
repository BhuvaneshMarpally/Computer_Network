**RIPv2 is a classless distance-vector routing protocol.**

Its key improvement:

> **RIPv2 includes the subnet mask in routing updates.**

Example advertisement:

```
Destination: 192.168.1.128
Subnet mask: 255.255.255.192
Metric: 1
Next hop: 0.0.0.0
```

The receiver directly learns:

```
192.168.1.128/26
```

There is no need to infer the mask from its own interface.

A next-hop field of `0.0.0.0` means the receiver should use the router that sent the update as the next hop.

### RIPv2 characteristics

- Uses **multicast `224.0.0.9`** for normal updates.
- Supports **VLSM**: different subnet masks within the same major network.
- Supports **CIDR** and classless route advertisements.
- Supports discontiguous networks when automatic summarization is disabled.
- Supports authentication.
- Can advertise summarized routes.

**Authentication verifies routing updates; it does not encrypt user traffic.**




| Feature                       | RIPv1                    | RIPv2                    |
| ----------------------------- | ------------------------ | ------------------------ |
| Protocol type                 | Distance vector          | Distance vector          |
| Routing behavior              | Classful                 | Classless                |
| Metric                        | Hop count                | Hop count                |
| Maximum reachable metric      | 15                       | 15                       |
| Unreachable metric            | 16                       | 16                       |
| Subnet mask in update         | No                       | Yes                      |
| VLSM support                  | No                       | Yes                      |
| CIDR support                  | No                       | Yes                      |
| Normal update destination     | `255.255.255.255`        | `224.0.0.9`              |
| Authentication                | No                       | Supported                |
| Periodic updates              | Approximately 30 seconds | Approximately 30 seconds |
| Transport                     | UDP                      | UDP                      |
| UDP port                      | 520                      | 520                      |
| Cisco administrative distance | 120                      | 120                      |