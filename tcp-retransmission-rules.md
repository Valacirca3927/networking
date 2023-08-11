The receiver does not request the retransmission. The sender waits for an ACK for the byte-range sent to the client and when not received, resends the packets, after a particular interval. 

**ARQ** - Automatic Repeat reQuest

There are several ways in which this is implemented:

http://stackoverflow.com/questions/12956685/what-are-the-retransmission-rules-for-tcp

> There's no fixed time for retransmission. Simple implementations estimate the RTT (round-trip-time)
 and if no ACK to send data has been received in 2x that time then they re-send.
>
> They then double the wait-time and re-send once more if again there is no reply. Rinse. Repeat.
>
> More sophisticated systems make better estimates of how long it should take for the ACK as well as
 guesses about exactly which data has been lost.
>
> The bottom-line is that there is no hard-and-fast rule about exactly when to retransmit. It's up
 to the implementation. All retransmissions are triggered solely by the sender based on lack of response from the receiver.

TCP never drops data so no, there is no way to indicate a server should forget about some segment.
