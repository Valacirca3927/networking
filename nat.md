SA = Source Address

DA = Destination Adress

```
                      INSIDE NETWORK                                   OUTSIDE NETWORK

           ┌────────────────────────────────────┐         ┌──────────────────────────────────────┐
           │                                    │         │                                      │
           │       ┌────────────┬─────────────┐ │         │       ┌─────────────┬──────────────┐ │
           │ ────► │    SA      │     DA      │ │ ──────► │ ────► │    SA       │     DA       │ │
  ┌──────┐ │       │Inside Local│Outside Local│ │         │       │Inside Global│Outside Global│ │ ┌───────┐
  │Inside│ │       └────────────┴─────────────┘ │  ┌───┐  │       └─────────────┴──────────────┘ │ │Outside│
  │ Host │ │                                    │  │NAT│  │                                      │ │ Host  │
  └──────┘ │ ┌────────────┬─────────────┐       │  └───┘  │ ┌─────────────┬──────────────┐       │ └───────┘
           │ │    SA      │     DA      │       │         │ │    SA       │     DA       │       │
           │ │Inside Local│Outside Local│ ◄──── │ ◄────── │ │Inside Global│Outside Global│ ◄──── │
           │ └────────────┴─────────────┘       │         │ └─────────────┴──────────────┘       │
           │                                    │         │                                      │
           └────────────────────────────────────┘         └──────────────────────────────────────┘
```

Based on a diagram [here.](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/13772-12.html)
