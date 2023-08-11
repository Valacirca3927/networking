[Understanding Single-Rate and Dual-Rate Traffic Policing - INE](https://ine.com/blog/2011-05-22-understanding-single-rate-and-dual-rate-traffic-policing)

> #### Dual Rate Traffic Contract
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