### Looking at CEF to see the default route on IOS

Cisco IOS really likes it when the mask is specified.
```
+------+            +------+
|  CE  +------------+  PE  |
+------+            +------+

        <-----------+
                Default Route
                  via BGP
```

#### RIB
```
CE# show ip route 0.0.0.0 0.0.0.0
Routing entry for 0.0.0.0/0, supernet
  Known via "bgp 2", distance 20, metric 0, candidate default path < --- BGP
  Tag 1, type external
  Last update from 172.16.0.1 00:09:55 ago
  Routing Descriptor Blocks:
  * 172.16.0.1, from 172.16.0.1, 00:09:55 ago   < --- Next Hop is *
      Route metric is 0, traffic share count is 1
      AS Hops 1
      Route tag 1
```
#### FIB
```
CE# show ip cef 0.0.0.0 0.0.0.0
0.0.0.0/0, version 9, epoch 0, cached adjacency 172.16.0.1
0 packets, 0 bytes
  via 172.16.0.1, 0 dependencies, recursive
    next hop 172.16.0.1, FastEthernet1/0 via 172.16.0.1/32  < -- interface & next-hop
    valid cached adjacency
```
#### Adjacency Table
```
CE# show adjacency fastEthernet 1/0 detail 
Protocol Interface                 Address
IP       FastEthernet1/0           172.16.0.1(7)
                                   0 packets, 0 bytes
                                   CA06154B001CC2081DDD00100800  < -- destination MAC & source MAC
                                   ARP        never     
                                   Epoch: 0
```
