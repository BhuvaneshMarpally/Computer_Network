
**RIPv1 is a classful distance-vector routing protocol.**

Its most important limitation:
> ***RIPv1 does not include a subnet mask in its routing updates.***
The receiving router must infer the mask.
### How does it infer the subnet mask?

For ordinary subnet advertisements:

| Situation                                                                                | How the receiver interprets the mask       |
| ---------------------------------------------------------------------------------------- | ------------------------------------------ |
| ==Advertised network belongs to the same classful major network as the receiving interface== | ==Uses the receiving interface's subnet mask== |
| ==Advertised network belongs to a different classful major network==                         | ==Uses the network's natural classful mask==   |

Natural classful masks:

|Class|First octet|Default mask|
|---|---|---|
|A|1–126|`/8` — `255.0.0.0`|
|B|128–191|`/16` — `255.255.0.0`|
|C|192–223|`/24` — `255.255.255.0`|


![](../../../../Screenshot%202026-09-28%20103647.png)
### Example: why the same `/26` mask can work

Suppose R1 receives an RIPv1 advertisement:

```
Destination: 192.168.1.128
Metric: 1
Subnet mask: Not included
```

R1's receiving interface is:

```
IP address: 192.168.1.1
Subnet mask: 255.255.255.192 (/26)
```

Both addresses belong to the classful major network `192.168.1.0/24`.

==Therefore, R1 uses its receiving interface's `/26` mask and interprets the route as:==

```
192.168.1.128/26
```

==**R1 did not receive `/26` from its neighbor. It inferred `/26` from its own interface configuration.**==

This works when the subnetting is consistent. If the advertised subnet actually uses a different mask, the inference can be wrong.

### RIPv1 characteristics

- Uses **broadcast `255.255.255.255`** for routing updates.
- Does not support VLSM or classless route advertisements.
- Automatically summarizes at classful network boundaries.
- Cannot correctly handle discontiguous subnet routing across a different major network.
- Does not support authentication.

**Classful does not mean “only `/8`, `/16`, or `/24` can be configured.”** RIPv1 can work with fixed-length subnetting, such as consistent `/26` subnets within a major network.


