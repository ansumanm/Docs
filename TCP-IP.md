# Table of contents
1. [TCP header](#tcp-header)
2. [Retransmission](#retransmission-of-missing-data----automatic-repeat-request-arq)
3. [Congestion Control](#congestion-control)

## TCP header

| Field                    | Size (bits) |
|--------------------------|-------------|
| Source Port              | 16          |
| Destination Port         | 16          |
| Sequence Number          | 32          |
| Acknowledgment Number    | 32          |
| Data Offset (Header Len) | 4           |
| Reserved                 | 6           |
| Flags                    | 6           |
| Window Size              | 16          |
| Checksum                 | 16          |
| Urgent Pointer           | 16          |
| Options (if any)         | Variable    |

### Source and Destination Port
Used to identify the endpoints within the source and destination machines

### Sequence Number(32) bits
Used for data sequencing. If the SYN flag is set, this is the initial sequence number.
The sequence number of the actual first data byte and the subsequent ones are used for data ordering, reassembly and loss recovery.

### Acknowledgement Number (32 bits)
If the ACK flag is set, the value of this field is the next sequence number that the sender of the packet is expecting.
This acknowledges the reciept of all prior bytes.

### Data Offset (Header Length)
Indicates the length of the TCP header in 32-bit words. It is needed because Optional fields might vary in length.

### Reserved (6 bits)
Should be set to 0

### Flags (6 bits)
Contains 9 different flags(URG, ACK, PSH, RST, SYN, FIN and 3 reserved bits)

### Window Size (16 bits)
#### Flow Control:
The size of the sender's receive window(or buffer space available), which controls the flow of data to prevent a receiver from being overwhelmed by too much data at once.

### Checksum (16 bits)
Used for error-checking of the header and data. It's calculated by the sender and verified by the receiver to ensure data integrity.

### Urgent Pointer (16 bits)
If the URG flag is set, this 16-bit field is an offset from the sequence number indicating the last urgent data byte.

### Options (variable):
The length of this field is determined by the data offset field. Options can control various aspects of the TCP connection, like maximum segment size, window scaling, selective acknowledgement etc...

## Retransmission of missing data -- Automatic Repeat reQuest (ARQ)
The process involves use of acknowledgements (ACKs) and timers.
### Positive Acknowledgement with Retransmission(PAR):
Each TCP segment is acknowledged by the receiver. If the sender doesn't receive an ACK for a particular segment within a certain time frame, it assumes that the segment was lost or corrupted and retransmits it.
### Selective Acknowledgements(SACK):
The receiver informs the sender about all the segments that have been received successfully. This way, the sender only needs to transmit the segments that have not been acknowledged, rather than transmitting all segments following a missing segment.
### Timers
#### Retransmission Timer:
Each time a segment is sent, a retransmission timer is started for a segment. If the timer expires before an acknowldegement is received, the segment is retransmitted.
#### Exponential Backoff:
To manage congestion and to avoid overwhelming the network or the receiver, the restransmission timer is often set to double after each consecutive timeout for the same segment, up to a certain limit.
#### Fast Retransmission
Another mechanism that triggers retransmission is the detection of three duplicate ACKs.
If the sender receives three ACKs for the same data(indicating that the segment immediately following the acknowledged data has not been received), it does not wait for the retransmission timer to expire before restransmitting the missing segment.

## Congestion Control
### Round Trip Time (RTT):
It represents the amount of time it takes for a data packet to travel from sender to the receiver and for an acknowledgement of that packet to return to the sender.
### Slow Start Phase
#### Initialization:
When a new TCP connection is established, Slow Start initializes the congestin window size **cwnd** to a small value, typically 1 or 2 segments.
#### Exponential increment:
Initially, congestion window (cwnd) = 1
After 1 RTT, cwnd = 2^(1) = 2
2 RTT, cwnd = 2^(2) = 4
3 RTT, cwnd = 2^(3) = 8

### Congestion Avoidance Phase
#### Additive increment
This phase starts after the threshold value **ssthresh** for the cwnd is reached. After that for each RTT, cwnd = cwnd + 1

### Congestion Detection Phase
#### Case 1: Retransmission due to Timeout
In this case, the congestion possibility is high.
- ssthresh is reduced to half of the current window size.
- set cwnd = 1
- start with the slow start phase again

#### Case 2:
Retransmission due to 3 Acknowledgement Duplicates - The congestion possibility is less.
- ssthresh value reduces to half of the current window size.
- set cwnd = ssthresh
- start with the congestion avoidance phase.





