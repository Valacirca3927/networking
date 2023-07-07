## 802.1D - Spanning Tree

The 802.1D committee wanted *two* learning states[^stp], one with and one without learning station addresses. This is why it's more complicated.

[^stp]: *Interconnections* - Radia Perlman, page 67.

```
+-----------+
|    off    |
+-----+-----+
      |
      |   Turn on interface
      v
+-----+-------+
|  Listening  | Recieve + Send BPDUs
+-----+-------+
      |
      |   forward delay (default 15s)
      v
+-----+------+
|  Learning  |  Recieve + Send BPDUs + Progam CAM
+-----+------+
      |
      |   forward delay (default 15s)
      v
+-----+--------+
|  Forwarding  | Recieve + Send BPDUs + Program CAM + Forward Frames
+--------------+
```

## ARP
Captured on-wire via GNS3 from ipterm-to-ipterm

```
packet #1 - who has 10.0.6.10? Tell 10.0.0.20
packet #2 - 10.0.0.10 is at ce:b1:5f:58:1d:8a
```
#### ARP Request
```
> Ethernet II

    Destination: Broadcast (ff:ff:ff:ff:ff:ff)
    Source: 1a:20:4e:9e:fb:9c (1a:20:4e:9e:fb:9c)
    Type: ARP (0x0806)

> Address Resolution Protocol (request)

    Hardware type: Ethernet (1)
    Protocol type: IPv4 (0x0800)
    Hardware size: 6
    Protocol size: 4
    Opcode: request (1)
    Sender MAC address: 1a:20:4e:9e:fb:9c (1a:20:4e:9e:fb:9c)
    Sender IP address: 10.0.0.20
    Target MAC address: 00:00:00_00:00:00 (00:00:00:00:00:00)
    Target IP address: 10.0.0.10
```


#### ARP Reply
```
> Ethernet II

    Destination: 1a:20:4e:9e:fb:9c (1a:20:4e:9e:fb:9c)
    Source: ce:b1:5f:58:1d:8a (ce:b1:5f:58:1d:8a)
    Type: ARP (0x0806)
    Padding: <lots of zeros>

> Address Resolution Protocol (reply)

    Hardware type: Ethernet (1)
    Protocol type: IPv4 (0x0800)
    Hardware size: 6
    Protocol size: 4
    Opcode: reply (2)
    Sender MAC address: ce:b1:5f:58:1d:8a (ce:b1:5f:58:1d:8a)
    Sender IP address: 10.0.0.10
    Target MAC address: 1a:20:4e:9e:fb:9c (1a:20:4e:9e:fb:9c)
    Target IP address: 10.0.0.20
```

### 802.1Q Frame Format

32 bits added to a ethernet frame to multiplex VLANs
```
                                   + Priority Code Point(PCP)
                                   |   Used for LAN CoS
                                   |
                                   |   +-+ Drop Elgible Indicator (DEI)
                                   |   |
                                   v   v
+-------------------------------+--+--+++-----------------------+
|    Tag Protocol Identifier    |     | |                       |
|     (TPID) Set to 0x8100      | PCP | |       VLAN ID         |
|                               |     | |                       |
|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8|1 2 3|4|5 6 7 8 1 2 3 4 5 6 7 8|
+---------------------------------------------------------------+
            16 bits                3   1        12 bits
```
| VLAN ID             | Purpose     | 
| -------------- | ---------------------|
|0| reserved for 802.1P|
|1| default vlan |
|2-1001| normal network operations|
|1002-1005| reserved|
|1006-4094| extended vlan range|
