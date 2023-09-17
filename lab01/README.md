#  Configure Router-on-a-Stick Inter-Vlan Routing
### Topology
![](Схема1.png)

###  Objectives

  1. Build the Network and Configure Basic Devise Settings;
  2. Create VLANs snd Assign Switch Ports;
  3. Configure an 802.1Q Trunk between the Switches;
  4. Configure Inter-VLAN Routing on the Router
  5. Verify Inter-VLAN Routing is working


Addresssing and VLAN Table

| Device     | Interface    | IP Address             | Subnet Mask         | Default Gateway   | Vlan         |Name |
|-----------------:|:---------------|-------------------------:|:--------------------|:-------------:|-----------------|-----------------|
| R1   | e0/0.3 | 192.168.3.1    | 255.255.255.0 |    |  |
| R1   | e0/0.4 | 192.168.4.1    | 255.255.255.0 |   |  |
| S1   | Vl3| 192.168.3.11    | 255.255.255.0 |192.168.3.1   | 3 |Management|
| S2   | Vl3| 192.168.3.12    | 255.255.255.0 |192.168.3.1   | 3 |Management|
| VPC4   | Vl3| 192.168.3.3    | 255.255.255.0 |192.168.3.1   | 3 |Management|
| VPC5   | Vl4| 192.168.4.3    | 255.255.255.0 |192.168.4.1   | 4 |Operations|

###  Step 1

  1. The network as shown in the topology was cabled;

###  Step 2

  1. The network as shown in the topology was cabled;