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

> custom_topo.py
```python
#!/usr/bin/python



from mininet.topo import Topo
from mininet.net import Mininet
from mininet.log import setLogLevel
from mininet.cli import CLI
from mininet.node import OVSSwitch, Controller, RemoteController
from time import sleep


class SingleSwitchTopo(Topo):
    def build(self):
        s1 = self.addSwitch('s1', protocols='OpenFlow13')
        s2 = self.addSwitch('s2', protocols='OpenFlow13')
        s3 = self.addSwitch('s3', protocols='OpenFlow13')
        s4 = self.addSwitch('s4', protocols='OpenFlow13')

        h1 = self.addHost('h1', mac="00:00:00:00:00:01", ip="10.0.0.1/24")
        h2 = self.addHost('h2', mac="00:00:00:00:00:02", ip="10.0.0.2/24")
        
        self.addLink(s1,s2,1,1)
        self.addLink(s1,s4,2,1) 
        self.addLink(s1,h1,3,1)
        self.addLink(s3,h2,3,1)
        self.addLink(s3,s4,2,2)
        self.addLink(s3,s2,1,2)

    


if __name__ == '__main__':
    setLogLevel('info')
    topo = SingleSwitchTopo()
    c1 = RemoteController('c1', ip='127.0.0.1')
    net = Mininet(topo=topo, controller=c1)
    net.start()
    CLI(net)
    net.stop()
```

2. Running The SDN Controller:
```sh
ryu-manager loadbalancer_project.py
```

3. Verifying the Flows installed on Switches using ovs commands (Optional).
```sh
sudo ovs-ofctl -O OpenFlow13 dump-flows s1
```

4. Testing Iperf client/Server on Hosts:
```sh
mininet> h2 iperf -u -s &
mininet> h1 iperf -c h2 -t 30 -P 7 &
```










[OpenNetworking]: <https://www.opennetworking.org >
