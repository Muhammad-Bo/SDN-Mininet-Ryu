### Groups

As [OpenNetworking] states:

The new group abstraction enables OpenFlow to represent a set of ports as a single entity for forwarding
packets. Different types of groups are provided, to represent different abstractions such as multicasting or
multipathing. Each group is composed of a set group buckets, each group bucket contains the set of actions
to be applied before forwarding to the port. Groups buckets can also forward to other groups, enabling to
chain groups together.

- Group indirection to represent a set of ports
- Group table with 4 types of groups:

  - All - used for multicast and flooding
  - Select - used for multipath
  - Indirect - simple indirection
  - Fast Failover - use first live port
  
- Group action to direct a flow to a group
- Group buckets contains actions related to the individual port

> Figure 1:

![Figure 1](https://wiki.onosproject.org/download/attachments/10560835/Screen%20Shot%202016-06-01%20at%206.07.35%20AM.png?version=1&modificationDate=1467912190741&api=v2)

> Figure 2:

![Figure 2](https://docs.pica8.com/download/thumbnails/3083229/of-group-abstract.png?version=1&modificationDate=1522250172000&api=v2)

### Project Specification

- Load Balancer Using Group Table 
- OpenFlow 1.3
- Mininet
- Ryu SDN Controller

![](https://raw.githubusercontent.com/Muhammad-Bo/SDN-Mininet-Ryu/master/Group%20Table%20-%20Bucket/load.png)


#### Run Ver 0.1

1. Running Custom Topolgy in Mininet
```sh
$ sudo python custom_topo.py
```













[OpenNetworking]: <https://www.opennetworking.org >
