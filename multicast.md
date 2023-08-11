Based on RFC 3973 Protocol Independent Multicast Dense Mode (PIM-DM)

PIM forms adjacencies in only one direction
PIM dense doesn't care about Designated Routers

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

## commands

#### IOS-XR
`show pim rpf hash`

`show mrib route summary`

#### IOS
`show ip rpf`
