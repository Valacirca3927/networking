#### BFD State Machine
Courtesy of the RFC

```
RFC 5880           Bidirectional Forwarding Detection          June 2010

(removed) 

The following diagram provides an overview of the state machine.
Transitions involving AdminDown state are deleted for clarity (but
are fully specified in sections 6.8.6 and 6.8.16).  The notation on
each arc represents the state of the remote system (as received in
the State field in the BFD Control packet) or indicates the
expiration of the Detection Timer.

                             +--+
                             |  | UP, ADMIN DOWN, TIMER
                             |  V
                     DOWN  +------+  INIT
              +------------|      |------------+
              |            | DOWN |            |
              |  +-------->|      |<--------+  |
              |  |         +------+         |  |
              |  |                          |  |
              |  |               ADMIN DOWN,|  |
              |  |ADMIN DOWN,          DOWN,|  |
              |  |TIMER                TIMER|  |
              V  |                          |  V
            +------+                      +------+
       +----|      |                      |      |----+
   DOWN|    | INIT |--------------------->|  UP  |    |INIT, UP
       +--->|      | INIT, UP             |      |<---+
            +------+                      +------+
```

* **BOB** - BFD over Bundle

* **BLB** - BFD over Logical Bundle - (VLANS, Sub-interfaces). This requires multipath to be enabled. Multipath doesn't inject BFD packets into the HP queue.

### IOS-XR Commands
```
multipath include location 0/1/CPU0
bundle coexistence bob-blb logical
show tech-support routing bfd file
```