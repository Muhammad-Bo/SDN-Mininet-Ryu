
Demonstrated: Phase 1 for TSN-SDN Integration

### Features

- Support Link Layer Discovery Protocol(LLDP, IEEE 802.1AB)
- Plot the Network Graph using Networkx library
- Multithreading Python

### Requirement
    python -m pip install -U pip setuptools
    python -m pip install matplotlib
    apt-get install python-tk
    pip install networkx
### Video Demonstration



https://user-images.githubusercontent.com/56366406/151682810-b85381db-5828-4536-a0bf-78d000bed3f0.mp4


### Running Mecahnism
- To be Added later

### Terminal 1
    sudo mn --controller=remote,ip=127.0.0.1 --mac -i 10.0.0.0/24 --switch=ovsk,protocols=OpenFlow13 --topo=linear,5
### Terminal 2
    ryu-manager --observe-links Network_Discovery.py 
### Terminal 1 (Mininet Shell)
    mininet> h1 ping -c 10 h2
