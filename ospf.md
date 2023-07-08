## Foundation

Each router in an area will have an identical `LSDB` Link state database. OSPF is two-tier, there is the backbone area `0`, and other areas have connectivity to the backbone.

## RFC
[RFC 2328 - OSPF Version 2](https://www.rfc-editor.org/rfc/rfc2328)

[Understand OSPF Design Guide - Cisco](https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/7039-1.html)

### Commands 
Command | Description                                                                         
-------------------------------------------- | -----------------------------------------------
`show ip ospf interface brief`               |  Shows all interfaces OSPF is running on       
`clear ip ospf process`                      |  Necessary to change the router-id.            
`show ip ospf int brief`                     |  Interfaces OSPF is enabled on                 
`show ip ospf int`                           |  OSPF interface information. Like network type 

### Packets

| Type  | Packet  name          | Protocol  function           |
| ----- | --------------------- | ---------------------------- |
| 1     | Hello                 | Discover/maintain  neighbors |
| 2     | Database Description  | Summarize database contents  |
| 3     | Link State Request    | Database download            |
| 4     | Link State Update     | Database update              |
| 5     | Link State Ack        | Flooding acknowledgment      |

[Via the RFC :)](https://www.rfc-editor.org/rfc/rfc2328#section-10.8)

#### Hello Packet

| Field | Description |
| ----- | ----------- | 
| Router ID | 32 bits. Usually an IP, Usually the loopback0 |
| Auth Options | `none`, `clear text`, `MD5` |
| Area ID | 32 bits. Can be in IP or just decimal. |
| Interface Address Mask | Subnet mask of the interface the hello is sent on |
| Interface Priority | What priority this interface is for a DR election |
| Hello Interval | Seconds between hello packets |
| Dead Interval | How long to wait in seconds before delaring a router down | 
| DR and BDR | IPs of the DR and BDR |
| Active Neighbor | List of neighbors seen on the network segment, within the Dead Interval | 

### LSAs

Default OSPF routing preference

intra-area > inter-area > external > nssa-external


OSPF LSA Type | Purpose | Notes
--- | --- | ---
1 | Locally Connected | Router
2 | Broadcast DR | Subnet
3 | ABR Summary | Inter-Area
4 | ASBR Summaries | - 


#### NSSA

`N` for NSSA

This is for an attached domain that isn't running OSPF, you want to redistribute into the domain, without allowing other Type 5s. 
The new LSA is type 7. Instead of E1 and E2 its N1 and N2.


##### Getting a default into a NSSA

- `area 2 nssa default-information-originate` 
- `area 2 default-cost 500`

Doesn't happen by default.

#### Totally Stubby

`area 3 stub no-summary`
- only done on the ABR
- filters out even inter-area routes

## Orginating a Default Route

OSPF has two ways of originating a default route.

`default-information originate` if a default route is present.

`default-information originate always` do it anyway.

## Authentication
	
#### Signing

1. OSPF builds the packet.
1. Sending router picks primary key
    - 8 bit key ID which specifies the hash and shared secret
1. Crytographic sequence number is added for replay protection (Typically seconds since boot)
1. Secret is appended to the packet
1. Router uses the packet and secret to hash the packet, replacing the secret with the hash
1. Packet gets transmitted

#### Verifying signature
1. Look at the Key ID
1. Hash the packet minus the pre-existing hash using the Key.
1. If it matches, verify the crypto sequence number is higher.
1. Accept.

## Network Types
 
[OSPF Representation of routers and networks](https://www.rfc-editor.org/rfc/rfc2328#page-13)

| CLI                                   |      Network Types      | Multicast or Unicast	| LSA Type 1 or 2	|   Use-case                                                                            |
| ------------------------------------- | ----------------------- | ----------------------- | ----------------- | ------------------------------------------------------------------------------------- |
| `ip ospf network broadcast`           | Broadcast		          | Multicast		        | 2 - DR Election	|   Ethernet, Token Ring, FDDI                                                          |
| `ip ospf network non-broadcast`       | NBMA[^NBMA]			  | **Unicast**[^unicast]   | 2 - DR Election 	|   X.25, frame-relay, ATM (poorly, the hub needs to be DR)                             |
| `ip ospf network point-to-point`      | point-to-point		  | Multicast		        | 1 - No DR		    |   T1, T3, Serial, HDLC, PPP (Full Adjacency)                                          |
| `ip ospf network point-to-multipoint` | point-to-multipoint	  | Multicast		        | 1 - No DR		    |   Frame-relay with a Hub router. Uses more resources then NBMA, more fault tollerant  |

[^NBMA]: Only John Moy understands this. It's very difficult to troubleshoot, but it is the RFC compliant (??) implementation. Use `ip ospf network point-to-multipoint`.
[^unicast]: The DR (which should be the HUB or bad things happen) needs to have static neighbor statements.

```
Moy                         Standards Track                    [Page 13]

RFC 2328                     OSPF Version 2                   April 1998

                                                  **FROM**

                                           *      |RT1|RT2|
                +---+Ia    +---+           *   ------------
                |RT1|------|RT2|           T   RT1|   | X |
                +---+    Ib+---+           O   RT2| X |   |
                                           *    Ia|   | X |
                                           *    Ib| X |   |

                     Physical point-to-point networks


                                                  **FROM**
                      +---+                *
                      |RT7|                *      |RT7| N3|
                      +---+                T   ------------
                        |                  O   RT7|   |   |
            +----------------------+       *    N3| X |   |
                       N3                  *

                              Stub networks

                                                  **FROM**
                +---+      +---+
                |RT3|      |RT4|              |RT3|RT4|RT5|RT6|N2 |
                +---+      +---+        *  ------------------------
                  |    N2    |          *  RT3|   |   |   |   | X |
            +----------------------+    T  RT4|   |   |   |   | X |
                  |          |          O  RT5|   |   |   |   | X |
                +---+      +---+        *  RT6|   |   |   |   | X |
                |RT5|      |RT6|        *   N2| X | X | X | X |   |
                +---+      +---+

                          Broadcast or NBMA networks
```

### Sham Link

**The Problem**

A customer with L3VPN service via OSPF-BGP-VPNv4 decides to connect two sites together via OSPF backdoor, a direct connection they manage themselves.<br>
When they turn on their private OSPF peering, all the traffic between these two sites now prefers the new link, vs the L3VPN cloud.

How can the environment be changed so traffic preferes the L3VPN but uses the OSPF backdoor link when the VPN is down? 

**The Solution: Sham Links**

Sham links are needed because the routes provided by an L3VPN are `O IA`. When the OSPF backdoor link comes up it will be preferred for two reasons:
* OSPF has a lower AD than BGP.
* `O` routes are prefered over `O IA`

A sham link makes two PE routers at different sites in the same customer VRF form an intra-area connection. 

From [OSPF Sham-Link Support for MPLS VPN - Cisco](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_ospf/configuration/15-sy/iro-15-sy-book/iro-sham-link.html#GUID-B0CBC9E8-D423-4AEF-BAB4-15FED3EA486C).

> Before you create a sham-link between PE routers in an MPLS VPN, you must:
>
> * Configure a new interface with a /32 address on the remote PE so that OSPF packets can be sent over the VPN backbone to the remote end of the sham-link. The /32 address must meet the following criteria:
>   * Belong to a VRF
>   * Not be advertised by OSPF
>   * Be advertised by BGP
>   * You can use the /32 address for other sham-links

###### R1 Config
```
!
! Can't use the main loopback.
! DO NOT ADVERTISE INTO OSPF
!
int loopback <number>
  desc ospf sham-Link endpoint
  ip vrf forwarding <vrf-name>
  ip address <ipv4> <mask>
!
! Customer OSPF Process
!
router ospf <process-id> vrf <vrf-name>
  area <area-id> sham-link <R1 source-address> <R2 destination-address> cost <number>
```

###### R2 Config
```
!
! Can't use the main loopback.
! DO NOT ADVERTISE INTO OSPF
!
int loopback <number>
  desc ospf sham-Link endpoint
  ip vrf forwarding <vrf-name>
  ip address <ipv4> <mask>
!
! Customer OSPF Process
!
router ospf <process-id> vrf <vrf-name>
  area <area-id> sham-link <R2 source-address> <R1 destination-address> cost <number>
```

#### Virtual Links

> the [OSPF Virtual Link](http://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/47866-ospfdb7.html) is slightly different, in ospf each are must connect to the backbone 0 directly and in real world scenarios this is not always possible so the virtual links allows you to bridge across another ospf are to connect to area 0
>
> -- Mark Malone [Cisco Community Answer](https://community.cisco.com/t5/routing/sham-link-vs-virtual-link/m-p/2892232/highlight/true#M266364)
