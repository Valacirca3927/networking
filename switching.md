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
