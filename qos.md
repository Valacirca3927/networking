[Understanding Single-Rate and Dual-Rate Traffic Policing - INE](https://ine.com/blog/2011-05-22-understanding-single-rate-and-dual-rate-traffic-policing)

#### Dual Rate Traffic Contract
> 
> The drawback of single-rate traffic contracts is that SP should be cautions assigning CIR bandwidth,
> and may effectively “undersell” itself, by offering less bandwidth than it can actually service at
> any given moment of type. The reason for this is the fact that not all customers send traffic simultaneously,
> so network links may effectively become underutilized even at the “weak spots”. This brings the idea
> of dual-rate traffic contract: supply customer with two sending rates, but only guarantee the smaller one.
> In case of congestion in the network, discard traffic that exceeds the committed rate more aggressively
> and signal the customer to slow down to the committed rate. This principle was first widely implemented
> in Frame-Relay networks, but could be easily replicated using any packet-switching technology. There are four main parameters in a dual-rate traffic contract.
>
> * **Committed information Rate (CIR)** - Same meaning as with a single-rate contract.
>  
> * **Committed burstsize (Bc)** - Same meaning as with a single rate contract, and once again,
>   Tc – the averaging interval – is implicitly defined as Tc=Bc/CIR.
> 
> * **Peak Information Rate (PIR)** - Additional parameter – defines the maximum average sending
>   rate for the customer. Traffic bursts that exceed CIR but remain under PIR are allowed in
>   the network, but marked for more aggressive discarding. Marking depends on the transport 
>   technology, e.g. it could be DSCP bits, ATM CLP or Frame-Relay DE bit.
>   
> * **Excessive BurstSize (Be)** - This value has different meaning, compared to a single-rate
>   contract. Be is the maximum size of the packet burstthat could be accepted to sustain the
>   PIR rate. Effectively, Be defines the second averaging interval, Te=Be/PIR, the averaging
>   rate for PIR metering. Keep in mind that just like with any packet networks packets are sent
>   at the AR, the actual physical rate – CIR and PIR are just average values.


## XR QoS Commands

###### Display all service-policies configured in a direction
```
show running interface * service-policy input
show running interface * service-policy output
```

###### Display QoS Statistics
```
show policy-map interface <int> input
show policy-map interface <int> output
```

###### Display details of a service-policy programmed in hardware
```
show qos interface <int> input
show qos interface <int> output
```

###### Display the interfaces a policy-map is attached to
```
show policy-map targets [pmap-name <name>] [location <loc>]
```

###### Debugging Commands
```
show tech qos
show app-obj db class_map_qos_db proc-name <name of DB proc> location <loc>
show app-obj db policy_map_qos_db proc-name <name of DB proc> location <loc>
```

## IOS Config
```
mls qos 

interface Vlan10
 no ip address
 service-policy input Ping_Set_DSCP

policy-map Interface_Police
  class port21
    police 96000 8000 exceed-action drop
  class port22
    police 64000 8000 exceed-action drop

policy-map Ping_Set_DSCP
  class Ping_Traffic
   set dscp af31
   service-policy Interface_Police

class-map match-all port22
  match input-interface FastEthernet0/22
class-map match-all port21
  match input-interface  FastEthernet0/21
class-map match-all Ping_Traffic
  match access-group 100

interface FastEthernet0/21
 mls qos vlan-based
```

## IOS-XR QoS Policy to shape outgoing to 100Mbps and police incoming to 100Mbps
```
interface GigabitEthernet0/0/0/0.100 l2transport
 service-policy input incoming-limit-to-100Mb
 service-policy output outgoing-limit-to-100Mb
!
policy-map incoming-limit-to-100Mb
 class class-default
  police rate 100 mbps
   exceed-action drop
  !
!
end-policy-map
!
policy-map outgoing-limit-to-100Mb
 class class-default
  shape average 100 mbps
 !
end-policy-map
```

#### Verification
```
show policy-map interface g0/0/0/0.100 input
show policy-map interface g0/0/0/0.100 output
```
