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
> loadbalancer_project.py
```python
# Copyright (C) 2011 Nippon Telegraph and Telephone Corporation.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from ryu.base import app_manager
from ryu.controller import ofp_event
from ryu.controller.handler import CONFIG_DISPATCHER, MAIN_DISPATCHER
from ryu.controller.handler import set_ev_cls
from ryu.ofproto import ofproto_v1_3
from ryu.lib.packet import packet
from ryu.lib.packet import ethernet
from ryu.lib.packet import ether_types


class SimpleSwitch13(app_manager.RyuApp):
    OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

    def __init__(self, *args, **kwargs):
        super(SimpleSwitch13, self).__init__(*args, **kwargs)
        self.mac_to_port = {}

    @set_ev_cls(ofp_event.EventOFPSwitchFeatures, CONFIG_DISPATCHER)
    def switch_features_handler(self, ev):
        datapath = ev.msg.datapath
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser

        # install table-miss flow entry
        #
        # We specify NO BUFFER to max_len of the output action due to
        # OVS bug. At this moment, if we specify a lesser number, e.g.,
        # 128, OVS will send Packet-In with invalid buffer_id and
        # truncated packet data. In that case, we cannot output packets
        # correctly.  The bug has been fixed in OVS v2.1.0.
        match = parser.OFPMatch()
        actions = [parser.OFPActionOutput(ofproto.OFPP_CONTROLLER,
                                          ofproto.OFPCML_NO_BUFFER)]
        self.add_flow(datapath, 0, match, actions)
        # We are checking the OVS1 Logic
        if datapath.id == 1:
            # Adding Group Table
            self.group_table(datapath)
            actions = [parser.OFPActionGroup(group_id=50)]
            match = parser.OFPMatch(in_port=3)  # Sending Host Packets to Group Table
            self.add_flow(datapath, 10, match, actions)

            actions = [parser.OFPActionOutput(3)]
            match = parser.OFPMatch(in_port=1)
            self.add_flow(datapath, 10, match, actions)

            actions = [parser.OFPActionOutput(3)]
            match = parser.OFPMatch(in_port=2)
            self.add_flow(datapath, 10, match, actions)

        # OVS 3 Logic
        if datapath.id == 3:
            self.group_table(datapath)
            actions = [parser.OFPActionGroup(group_id=50)]
            match = parser.OFPMatch(in_port=3)
            self.add_flow(datapath, 10, match, actions)

            actions = [parser.OFPActionOutput(3)]
            match = parser.OFPMatch(in_port=1)
            self.add_flow(datapath, 10, match, actions)

            actions = [parser.OFPActionOutput(3)]
            match = parser.OFPMatch(in_port=2)
            self.add_flow(datapath, 10, match, actions)

    def add_flow(self, datapath, priority, match, actions, buffer_id=None):
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser

        inst = [parser.OFPInstructionActions(ofproto.OFPIT_APPLY_ACTIONS,
                                             actions)]
        if buffer_id:
            mod = parser.OFPFlowMod(datapath=datapath, buffer_id=buffer_id,
                                    priority=priority, match=match,
                                    instructions=inst)
        else:
            mod = parser.OFPFlowMod(datapath=datapath, priority=priority,
                                    match=match, instructions=inst)
        datapath.send_msg(mod)

    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    def _packet_in_handler(self, ev):
        # If you hit this you might want to increase
        # the "miss_send_length" of your switch
        if ev.msg.msg_len < ev.msg.total_len:
            self.logger.debug("packet truncated: only %s of %s bytes",
                              ev.msg.msg_len, ev.msg.total_len)
        msg = ev.msg
        datapath = msg.datapath
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser
        in_port = msg.match['in_port']

        pkt = packet.Packet(msg.data)
        eth = pkt.get_protocols(ethernet.ethernet)[0]

        if eth.ethertype == ether_types.ETH_TYPE_LLDP:
            # ignore lldp packet
            return
        dst = eth.dst
        src = eth.src

        dpid = datapath.id
        self.mac_to_port.setdefault(dpid, {})

        self.logger.info("packet in %s %s %s %s", dpid, src, dst, in_port)

        # learn a mac address to avoid FLOOD next time.
        self.mac_to_port[dpid][src] = in_port

        if dst in self.mac_to_port[dpid]:
            out_port = self.mac_to_port[dpid][dst]
        else:
            out_port = ofproto.OFPP_FLOOD

        actions = [parser.OFPActionOutput(out_port)]

        # install a flow to avoid packet_in next time
        if out_port != ofproto.OFPP_FLOOD:
            match = parser.OFPMatch(in_port=in_port, eth_dst=dst, eth_src=src)
            # verify if we have a valid buffer_id, if yes avoid to send both
            # flow_mod & packet_out
            if msg.buffer_id != ofproto.OFP_NO_BUFFER:
                self.add_flow(datapath, 1, match, actions, msg.buffer_id)
                return
            else:
                self.add_flow(datapath, 1, match, actions)
        data = None
        if msg.buffer_id == ofproto.OFP_NO_BUFFER:
            data = msg.data

        out = parser.OFPPacketOut(datapath=datapath, buffer_id=msg.buffer_id,
                                  in_port=in_port, actions=actions, data=data)
        datapath.send_msg(out)

    def group_table(self, datapath):
        ofproto = datapath.ofproto
        parser = datapath.ofproto_parser

        weight_1 = 50
        weight_2 = 50

        watching_port_getting = ofproto_v1_3.OFPP_ANY
        watching_group_getting = ofproto_v1_3.OFPQ_ALL

        actions1 = [parser.OFPActionOutput(1)]
        actions2 = [parser.OFPActionOutput(2)]
        buckets = [parser.OFPBucket(weight_1, watching_port_getting, watching_group_getting, actions=actions1),
                   parser.OFPBucket(weight_2, watching_port_getting, watching_group_getting, actions=actions2)]
        
        req = parser.OFPGroupMod(datapath, ofproto.OFPGC_ADD,
                                 ofproto.OFPGT_SELECT, 50, buckets)
        datapath.send_msg(req)
```

3. Verifying the Flows installed on Switches using ovs commands (Optional).
```sh
sudo ovs-ofctl -O OpenFlow13 dump-flows s1
```

4. Testing Iperf client/Server on Hosts:
```sh
mininet> h2 iperf -s &
mininet> h1 iperf -c h2 -t 50 -P 6 &
```










[OpenNetworking]: <https://www.opennetworking.org >
