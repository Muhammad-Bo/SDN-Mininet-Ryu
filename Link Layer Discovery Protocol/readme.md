
Demonstrated: UT-NETWORK LAB

### Features

- Support Link Layer Discovery Protocol(LLDP, IEEE 802.1AB)
- Plot the Network Graph using Networkx library
- Multithreading Python

### Requirement
    apt-get install python-tk
    pip install networkx
    
### Running Mecahnism
- To be Added later

### Terminal 1
    sudo mn --controller=remote,ip=127.0.0.1 --mac -i 10.0.0.0/24 --switch=ovsk,protocols=OpenFlow13 --topo=linear,5
### Terminal 2
    ryu-manager --observe-links Network_Discovery.py 
### Terminal 1 (Mininet Shell)
    mininet> h1 ping -c 10 h2
