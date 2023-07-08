## Root bridges election in Spanning Tree.

Two bridges send each other BPDUs, they compare bridge IDs to see who will keep sending BPDUs

The bridge with the lower ID (priority + mac address) wins. The non-root-bridge copies this bridge ID into it's BPDU, and sends that downstream.

The default for priority is `32768` or `0x80` on the wire. Because the 802.1D committee exists, the priority is this, plus the vlan ID.

**Always** configure a root bridge, or the *oldest device* with probably the *lowest mac address* wins the root bridge election.

```
switch(config)#spanning-tree vlan 60 priority ?
% Bridge Priority must be in increments of 4096.
% Allowed values are: 
  0     4096  8192  12288 16384 20480 24576 28672
  32768 36864 40960 45056 49152 53248 57344 61440
```
## Port Types

**Designated ports** send BPDUs downstream.

**Root Ports** are the best port towards the root bridge, either the lowest total cost or the lowest advertised priority or lowest advertised port ID (interface number).

## BPDU Fields

**Root ID** - The bridge that has won and is winning the elections

**Bridge ID** - The bridge sending the BPDUs.

## Root Path Cost
**Root Path Cost** - What the interfaces costs + the advertised cost to the root. The root sends a cost of 0.

### STP Path Calculations
`spanning-tree pathcost method long`

| Speed      | Short-Mode Cost | Long-Mode Cost |
|------------|-----------------|----------------|
| 10 Mbps    | 100             | 2000000        |
| 100 Mbps   | 19              | 200000         |
| 1 Gbps     | 4               | 20000          |
| 10 Gbps    | 2               | 2000           |
| 20 Gbps    | 1               | 1000           |
| 40 Gbps    | 1               | 500            |
| 100 Gbps   | 1               | 200            |
| 1 Tbps     | 1               | 20             |
| 10 Tbps    | 1               | 2              |













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

#### BPDU Frame Format

This is a RSTP BPDU in Packet Tracer
```
Spanning Tree Protocol

    Protocol Identifier: Spanning Tree Protocol (0x0000)
    Protocol Version Identifier: Rapid Spanning Tree (2)
    BPDU Type: Rapid/Multiple Spanning Tree (2x02)
    BPDU flags: 0x3c, Forwarding, Learning, Port Role: Designated
    
    0... .... = Topology Change Acknowledgment: No
    .0.. .... = Agreement: No
    ..1. .... = Forwarding: Yes
    ...1 .... = Learning: Yes
    .... 11.. = Port Role: Designated (3)
    .... ..0. = Proposal: No
    .... ...0 = Topology Change: No
    
    Root Identifier: 32768 / 1 / aa:bb:cc:00:07:00
    Root Path Cost: 100
    
    Bridge Identifier: 32768 / 1 / aa:bb:cc:00:0a:00
    Port identifier: 0x8003
    Message Age: 1
    Max Age: 20
    Hello Time: 2
    Forward Delay: 15
    Version 1 Length: 0
```

This is what the BPDU looks like on-the-wire

```
+-------------------------------+---------------+---------------+
|                               |               |               |
|          Protocol ID          |    Version    |   BPDU Type   |
|                               |               |               |
|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8|
+-------------------------------+---------------+---------------+
             2 bytes                  1 byte         1 byte

+---------------+----------------------------------------------->
|               |
|     Flag      |                    Root ID
|               |
|1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8
+---------------+----------------------------------------------->
    1 byte                            8 bytes

<--------------------------------------------------------------->

                           Root ID

 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8
<--------------------------------------------------------------->
                          8 bytes

<---------------+----------------------------------------------->
                |
    Root ID     |              Root Path Cost
                |
 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8
<---------------+----------------------------------------------->
    8 bytes                       4 bytes

<---------------+----------------------------------------------->
 Root Path Cost |
                |                Bridge ID
                |
 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8
<---------------+----------------------------------------------->
  4 bytes                         8 bytes

<--------------------------------------------------------------->

                           Bridge ID

 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8
<--------------------------------------------------------------->
                          8 bytes

<---------------+-------------------------------+--------------->
                |                               | Message age
   Bridge ID    |           Port ID             |  (in 1/256s of a second)
                |                               |
 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8
<---------------+-------------------------------+--------------->
    8 bytes                2 Bytes                   2 Bytes

<---------------+-------------------------------+--------------->
                |           Max Age             | Hello Time
   Message Age  |        (in 1/256ths)          |  (in 1/256ths of a second)
                |                               |
 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8
<---------------+-------------------------------+--------------->
    2 Bytes           2 Bytes                        2 Bytes

<---------------+-------------------------------+---------------+
                |  Forward Delay                |   Version 1   |
   Hello Time   |    (in 1/256ths of a second)  |    Length     |
                |                               |               |
 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8|1 2 3 4 5 6 7 8|
<---------------+-------------------------------+---------------+
    2 Bytes                 2 Bytes                   1 Byte

+-------------------------------+
|                               |
|      Version 3 Length         |
|                               |
|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8|
+-------------------------------+
           2 Bytes
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
+-------------------------------+-----+-+-----------------------+
|    Tag Protocol Identifier    |     | |                       |
|     (TPID) Set to 0x8100      | PCP | |       VLAN ID         |
|                               |     | |                       |
|1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8|1 2 3|4|5 6 7 8 1 2 3 4 5 6 7 8|
+-------------------------------+-----+-+-----------------------+
            16 bits                3   1        12 bits
```
| VLAN ID             | Purpose     | 
| -------------- | ---------------------|
|0| reserved for 802.1P|
|1| default vlan |
|2-1001| normal network operations|
|1002-1005| reserved|
|1006-4094| extended vlan range|
