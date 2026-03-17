High Level Approach:

Designed like TCP Reno, including base TCP protocol features:
- Sequence numbers
- Out-of-order packet checking
- Sliding Window
and additional features:
- Fast Retransmission
- Fast Recovery

Sliding Window:
The sliding window works by conditionally adding stdin into sockets only if window is not full, by maintaining a lowest_unacked and only sends while the seq < lowest_unacked + wnd. When the receiver replies with an ACK, it is assumed that all packets up to that ack number are all ACKed. Thus we can clear out our list of unacked segments and move the window forward.

Change to Sender socket shutdown:
After EOF on stdin the sender will stop reading stdin and wait for any remaning ACKs from receiver until exiting.

Out-of-order packets:
The receiver keeps track of buffered packets that are supposed to come after missing packets. However, the stdout must print data in order. So if an out of order SEQ is sent by the sender, then the receiver populates the buffer with said out of order packet, to be printed out later when the missing packet is received. When the missing one is received, we flush the buffer out and print all subsequent packets after the missing one finally arrived. The receiver also sends ACK for the last in order seq just like dupe packs to make fast retransmit work. 

Duplicate packets:
If the receiver gets a duped seq, it will reACK the last segment confirmed to have been received. Dupes are not printed.

Retransmission:
The send tracks the send time of each unacked packet when sent. If the oldest unacked packet exceeds 1s (assumed RTT), then initiate retransmits all unacked packets and refreshers their send time. Also retransmits if the sender sees 3 dupe ACKs in a row for the same packet. 
