## Theory

Multicast is always TO a group not FROM a group. 

A multicast address can only ever be a destination.

Scope          | Address
-------------- | --------------
`224.0.0.0/4`  | Multicast
`240.0.0.0/24` | Link-Local
`232.0.0.0/8`  | Source Specific (SSM)
`239.0.0.0/8`  | Private or Administratively Scoped


Based on RFC 3973 Protocol Independent Multicast Dense Mode (PIM-DM)

PIM forms adjacencies in only one direction

PIM dense doesn't care about Designated Routers

The multicast source is the root of the tree. Packets flow downstream from the source. Control plane traffic like PIM joins flow upstream to the RP, or to the reciever.

## PIM Dense Mode

- Push or Implicit Join
  - Flood and Prune
  - Routers with no Recievers prune
  - `224.0.0.13` to find neighbors
  - Send traffic out all interfaces running dense
  - Recievers prune back
  - Router attached to LAN listens for multicast control plane.
     - Recieves source traffic
       - Insert (*,G) and (S,G) into mrib
       - Incomming traffic is attached to IIL
       - OIL is all other interfaces
       - Flood to OIL
       - PIM dense always uses SPT.
- Prune occurs
  - Traffic flows stop, but (S,G) remains in table
  - Multicast fails RPF
  - No downstream neighbor or reciever
  - Downstream sent prune
  - LAN Prune override exception

- After pruning 
  - Flood again, prune back, flood again, prune back
  
## PIM Sparse
- Discover PIM neighbors
- Discover the RP
- Tell RP about the source
- Tell RP about recievers 
- Build shortest path tree from sender to recievers through RP
- Join shortest path tree
- Leave shared tree
- Perform mrib maintainance

## Commands

#### IOS-XR
`show pim rpf hash`

`show pim range-list`

`show pim topology`

`show ip mroute`

`show mrib route summary`

#### IOS
`show ip rpf`

#### Nexus 7K
`show forwarding multicast route group <>`
