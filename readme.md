High Level Approach:

Designed like TCP Reno, including base TCP protocol features:
- Sequence numbers
- Out-of-order packet checking
- Sliding Window
and additional features:
- Fast Retransmission
- Fast Recovery

Out-of-order packets:
The receiver keeps track of buffered packets that are supposed to come after missing packets.
However, the stdout must print data in order. So if an out of order SEQ is sent by the sender,
then the receiver populates the buffer with said out of order packet, to be printed out later
when the missing packet is received. When the missing one is received, we flush the buffer
out and print all subsequent packets after the missing one finally arrived.