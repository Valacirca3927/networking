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
