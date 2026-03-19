High Level Approach:

Designed like TCP Reno, including base TCP protocol features:
- Sequence numbers
- Out-of-order packet checking
- Sliding Window
and additional features:
- Congestion Control (cwnd, ssthresh, slow start, congestion avoidance)
- Fast Retransmission
- Fast Recovery
- RTT Estimation
- Checksum Verification

Sliding Window:
The sliding window works by conditionally adding stdin into sockets only if window is not full, by maintaining a lowest_unacked and only sends while the seq < lowest_unacked + cwnd. When the receiver replies with an ACK, it is assumed that all packets up to that ack number are all ACKed. Thus we can clear out our list of unacked segments and move the window forward.

Congestion Control:
We use cwnd and ssthresh to control the window size. cwnd starts at 1 and ssthresh starts at 64. During slow start, cwnd grows by 1 for each new ACK received (doubles every RTT). Once cwnd reaches ssthresh, we switch to congestion avoidance and grow cwnd by 1/cwnd per ACK (linear growth). On a timeout we set ssthresh to cwnd/2 and reset cwnd back to 1 to re-enter slow start.

Fast Retransmission and Fast Recovery:
On 3 duplicate ACKs we enter fast recovery. We set ssthresh to cwnd/2, set cwnd to ssthresh + 3, and immediately retransmit the lost packet. Each additional dupe ACK increases cwnd by 1. When we get a new ACK that advances the window we exit fast recovery and set cwnd back to ssthresh.

RTT Estimation:
We measure the round trip time using EWMA smoothing and we set the retransmission timeout to 2x the smoothed RTT with a  floor of 0.25s. We only sample RTT from packets that havent been retransmitted (Karn's) to avoid messing with the estimate.

Checksum:
Every packet (data and ACKs) includes an checksum computed with the packet contents. On receive we verify the checksum and drop any packet that fails verification.

Out-of-order packets:
The receiver keeps track of buffered packets that are supposed to come after missing packets. However, the stdout must print data in order. So if an out of order SEQ is sent by the sender, then the receiver populates the buffer with said out of order packet, to be printed out later when the missing packet is received. When the missing one is received, we flush the buffer out and print all subsequent packets after the missing one finally arrived. The receiver also sends ACK for the last in order seq just like dupe packs to make fast retransmit work. 

Duplicate packets:
If the receiver gets a duped seq, it will reACK the last segment confirmed to have been received. Dupes are not printed.

Challenges:
The overall architecture and implementation for this project wasn't very challenging, the main challenge was oftentimes in the details for the congestion control etc, having to deal with complex branching functions and cases was difficult, but what helped was modeling after the math and formulas of the original features like congestion control. By dividing and conquering the work and tackling things function to function we were able to complete our implementation.

Testing:
We tested using the provided run and test script against all config files for levels 1-8. We ran individual configs to debug specific failure and checked the output to tune performance, and did the entire test suite to check the overall program.
