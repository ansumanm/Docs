# Table of contents
1. [TCP header](#tcp-header)
2. [Retransmission](#retransmission-of-missing-data----automatic-repeat-request-arq)
3. [Congestion Control](#congestion-control)
4. [Setsocketopt](#setsockopt)
5. [TCP Implementation Overview](#tcp-implementation-overview)
6. [TCP connection states](#tcp-connection-states)
7. [TCP connection open 3-way handshake](#tcp-connection-open-3-way-handshake)
8. [TCP connection close 4-way handshake](#tcp-connection-close-4-way-handshake)
9. [TCP header options](#tcp-header-options)
10. [Maximum Segment Size](#maximum-segment-size)
11. [Path MTU Discovery](#path-mtu-discovery)
12. [Timestamps](#timestamps)

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
| Flag | Name               | Description                                                                                             | Use Case                                                                                     |
|------|--------------------|---------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| URG  | Urgent             | Indicates that the Urgent pointer field is significant and that data should be processed immediately.  | Used for data that should be prioritized, such as interrupts or immediate action requests.   |
| ACK  | Acknowledgment     | Acknowledges receipt of data. Must be set in all packets except the initial SYN packet.                | Essential for the reliable delivery mechanism of TCP, confirming receipt of data.            |
| PSH  | Push               | Instructs the receiving host to push this data to the application as soon as possible.                 | Ideal for interactive applications requiring low latency, like telnet or SSH.                |
| RST  | Reset              | Resets the connection due to an error or because a packet was received for a non-existent connection.  | Used to abort a connection if a serious error occurs or to reject invalid packets.           |
| SYN  | Synchronize        | Used during the initial connection setup phase to synchronize sequence numbers.                        | Critical for establishing a TCP connection, initiating sequence number synchronization.     |
| FIN  | Finish             | Indicates that the sender has finished sending data and wishes to terminate the connection.           | Used to gracefully close a TCP connection after the data transfer is complete.               |


### Window Size (16 bits)
#### Flow Control:
The size of the sender's receive window(or buffer space available), which controls the flow of data to prevent a receiver from being overwhelmed by too much data at once.

### Checksum (16 bits)
Used for error-checking of the header and data. It's calculated by the sender and verified by the receiver to ensure data integrity.

This is how checksum is calculated:-
1. A pseudo header is constructed

#### IPv4 Pseudo Header Format

| Field              | Size (bits) | Description                                |
|--------------------|-------------|--------------------------------------------|
| Source Address     | 32          | The IPv4 address of the sender.            |
| Destination Address| 32          | The IPv4 address of the receiver.          |
| Zeros              | 8           | Reserved field set to zero.                |
| Protocol           | 8           | Protocol number for TCP (6).               |
| TCP Length         | 16          | Length of the TCP header and data in bytes.|

#### IPv6 Pseudo Header Format

| Field              | Size (bits)  | Description                                         |
|--------------------|--------------|-----------------------------------------------------|
| Source Address     | 128          | The IPv6 address of the sender.                     |
| Destination Address| 128          | The IPv6 address of the receiver.                   |
| TCP Length         | 32           | Length of the TCP header and data in bytes.         |
| Zeros              | 24           | Reserved for future use, set to zero.               |
| Next Header        | 8            | Identifies the protocol used (TCP is represented as 6).|

2. **Preparation for Checksum Calculation**
The pseudo header, the TCP header, and the TCP data are arranged in a sequence of 16-bit words for the purpose of checksum calculation.

3. **Checksum Computation**: The checksum algorithm sums all 16-bit words using 1's complement arithmetic. The sum itself is also represented as a 16-bit word and its 1's complement is taken to producte the final checksum value.

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

## Setsockopt
List of some common socket options that can be passed to `setsockopt()`, along with a brief description of each:

| Level             | Option                       | Description                                                                                   |
|-------------------|------------------------------|-----------------------------------------------------------------------------------------------|
| `SOL_SOCKET`      | `SO_REUSEADDR`               | Allows the socket to bind to an address/port in use.                                          |
| `SOL_SOCKET`      | `SO_KEEPALIVE`               | Keeps connections active by enabling keep-alive messages.                                     |
| `SOL_SOCKET`      | `SO_BROADCAST`               | Allows the sending of broadcast messages.                                                     |
| `SOL_SOCKET`      | `SO_SNDBUF`                  | Sets the send buffer size for the socket.                                                     |
| `SOL_SOCKET`      | `SO_RCVBUF`                  | Sets the receive buffer size for the socket.                                                  |
| `SOL_SOCKET`      | `SO_LINGER`                  | Specifies actions for unsent data when the socket is closed.                                  |
| `SOL_SOCKET`      | `SO_OOBINLINE`               | Allows out-of-band data to be received in the normal data stream.                             |
| `SOL_SOCKET`      | `SO_RCVTIMEO`                | Sets the timeout value that specifies the maximum amount of time a call to `recv()` can block.|
| `SOL_SOCKET`      | `SO_SNDTIMEO`                | Sets the timeout value specifying the maximum amount of time a call to `send()` can block.    |
| `IPPROTO_IP`      | `IP_TTL`                     | Sets the time-to-live field of IP packets.                                                    |
| `IPPROTO_IP`      | `IP_MULTICAST_TTL`           | Sets the TTL for multicast packets.                                                           |
| `IPPROTO_IP`      | `IP_MULTICAST_LOOP`          | Controls loopback of multicast packets.                                                       |
| `IPPROTO_IP`      | `IP_ADD_MEMBERSHIP`          | Adds the socket to the multicast group.                                                       |
| `IPPROTO_IP`      | `IP_DROP_MEMBERSHIP`         | Drops the socket from the multicast group.                                                    |
| `IPPROTO_IPV6`    | `IPV6_V6ONLY`                | Restricts the socket to IPv6 traffic only.                                                    |
| `IPPROTO_IPV6`    | `IPV6_JOIN_GROUP`            | Joins the specified IPv6 multicast group.                                                     |
| `IPPROTO_IPV6`    | `IPV6_LEAVE_GROUP`           | Leaves the specified IPv6 multicast group.                                                    |
| `IPPROTO_IPV6`    | `IPV6_MULTICAST_HOPS`        | Sets the hop limit for IPv6 multicast packets.                                                |
| `IPPROTO_TCP`     | `TCP_NODELAY`                | Disables the Nagle algorithm to send small packets immediately.                               |
| `IPPROTO_TCP`     | `TCP_MAXSEG`                 | Specifies the maximum segment size.                                                           |
| `IPPROTO_TCP`     | `TCP_KEEPIDLE`               | Sets the timeout value before the first keepalive probe is sent.                              |
| `IPPROTO_TCP`     | `TCP_KEEPINTVL`              | Sets the interval between successive keepalive probes.                                        |
| `IPPROTO_TCP`     | `TCP_KEEPCNT`                | Sets the number of keepalive probes before a connection is considered dropped.                |

This table provides an overview of some of the options you can set with `setsockopt()` to customize the behavior of your sockets in a POSIX environment.

## TCP Implementation Overview
- ### Protocol registration:
  TCP, like other network protocols in the Linux kernel, is registered in the networking subsystem using a protocol family-specific structure
  (**inet_connection_sofk_af_ops** for IPv4, for example). The registration process includes pointers to functions that handle various aspects of the protocol, such as socket creation, state transitions and data transmission/reception.
- ### Socket Layer Integrations:
  At user levet, applications interact with TCP through sockets using the socket API:-
    - **socket()**
    - **connect()**
    - **listen()**
    - **accept()**
    - **send()**
    - **recv()**
- ### TCP Control Blocks:
  The main data structures used to manage TCP connections are:-
    - **sock(or socket)**: Represents and endpoint for communication.
    - **inet_sock**: Inherits from **sock** and is used for IP-based sockets, adding fields specific to UP protocols like IP addresses and ports.
    - **tcp_sock**: Further specializes inet_sock for TCP, containing TCP-specific information such as sequence numbers, window sizes and various flags indicating the state of the TCP connection.
- ### TCP State Machine:
  Different states of TCP connection are
  - **LISTEN**
  - **SYN_SENT**
  - **SYN_RECEIVED**
  - **ESTABLISHED**
  - **FIN_WAIT_1**
  - **FIN_WAIT_2**
  - **CLOSE_WAIT**
  - **CLOSING**
  - **CLOSED**
  - **LAST_ACK**
  - **TIME_WAIT**
  - **BOUND**
- ### Buffers and Queues:
  - #### Send Buffer
  - #### Receive Buffer
  - #### Out-of-Order Queue
- ### Congestion Control and Flow Control
  Implementation includes algorithms for congestion control and flow control. These mechanisms adjust the rate of data transmission based on network conditions and receiver capabilities to optimize throughput and prevent congestion.
- ### Timers
  Various timers used to manage retransmission, detect dead connections etc.

## TCP connection states

| State             | Description                                                                                   |
|-------------------|-----------------------------------------------------------------------------------------------|
| `CLOSED`          | Indicates that the connection is not currently in use.                                        |
| `LISTEN`          | Waiting for a connection request from any remote TCP and port.                                |
| `SYN_SENT`        | A connection request has been sent; waiting for acknowledgment.                               |
| `SYN_RECEIVED`    | Initial synchronization (SYN) request received and responded with SYN-ACK; awaiting ACK.      |
| `ESTABLISHED`     | The connection has been established; data can be sent and received.                           |
| `FIN_WAIT_1`      | The application has requested to close the connection; waiting for the remote TCP to acknowledge the request. |
| `FIN_WAIT_2`      | The first FIN has been acknowledged; waiting for the remote TCP's connection termination request. |
| `CLOSE_WAIT`      | The remote TCP has initiated a connection termination request; waiting for the application to request to close the connection. |
| `CLOSING`         | Both sides have initiated a connection termination but neither side has completed it yet.     |
| `LAST_ACK`        | Waiting for the last acknowledgment after sending a termination request and receiving a FIN.  |
| `TIME_WAIT`       | Waiting for enough time to pass to ensure the remote TCP received the acknowledgment of its connection termination request. |
| `BOUND`           | Optional state used to indicate that the socket is bound to an address but not yet in a state where it is ready to connect or listen. (Not always considered a distinct TCP state in all implementations.) |

These states are part of the TCP finite state machine, which governs the lifecycle of a TCP connection from initiation through data transfer and finally to termination. Understanding these states is crucial for diagnosing network issues and for developing network applications.

## TCP connection open 3-way handshake
- ### SYN SENT
  Client sends a TCP segment with the SYN buti set to 1. This segment also includes an initial sequence number chosen by the sender to start the sequence numbers of the segments it will send.
- ### SYN-ACK Received
  The server receives the SYN request, responds with a TCP segment that has both the SYN and ACK bits set to 1. The ACK bit acknowledges the sender's SYN packet by including an acknowledgement number that is one more than the received initial sequence number. The receiver also sets it ISN.
- ### ACK Sent
  The client receives the SYN-ACK packet and responds with a TCP segment where the ACK bit is set to 1. This acknowledgement packet includes and acknowledgement number that is one more that the received initial sequence number from the receiver.

## TCP connection close 4-way handshake
- ### FIN from Client
  Client sends a TCP segment with FIN flag set to 1.
- ### ACK from Server
  The server sends ACK.
- ### FIN from Server
  The server sends FIN segment to client
- ### Ack from Client
  THe client sends ACK to acknowledge the FIN.The client goes to TIME_WAIT state  ( usually a duration of 2*MSL - Maximum Segment Lifetime) to ensure all packets have been properly received and to avoid potential conflicts with the new connection.

## TCP header options
Here's an expanded table that includes both the previously mentioned TCP options and some additional ones, capturing a broader range of TCP header options for enhanced functionalities and optimizations:

| Option                         | Description                                                                                                                                                              | Use Case                                                                 |
|--------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------|
| Maximum Segment Size (MSS)     | Specifies the max amount of data in a single TCP segment.                                                                                                               | Optimizes segment size to prevent IP fragmentation.                     |
| Window Scale (WSopt)           | Scales the window size field to allow for larger window sizes.                                                                                                           | Enhances data transmission efficiency in high-speed networks.           |
| Selective Acknowledgment (SACK)| Allows acknowledging non-contiguous blocks of data.                                                                                                                      | Improves efficiency in lossy conditions by reducing unnecessary retransmissions. |
| Timestamps                     | Supports RTT measurements and protects against sequence number wrapping (PAWS).                                                                                         | Enables accurate RTT calculation and handles sequence number issues in long-lived connections. |
| No-Operation (NOP)             | Used as padding to align options on 32-bit boundaries or as filler.                                                                                                      | Facilitates option alignment without conveying specific functionality.  |
| End of Option List (EOOL)      | Marks the end of the options list in the TCP header.                                                                                                                     | Indicates where options end and payload data begins.                    |
| Experimental (EXP)             | Reserved for experimental use, identified by specific kinds.                                                                                                             | Used for testing new features not intended for general use.             |
| Quick-Start Response (QS)      | Allows TCP connections to request and use higher initial sending rates.                                                                                                  | Reduces time to ramp up to full speed in high-capacity networks (experimental). |
| User Timeout Option (UTO)      | Configures the user timeout value for unacknowledged data.                                                                                                               | Defines the maximum time before a connection is forcibly closed due to unacknowledged data. |
| TCP Authentication Option (TCP-AO) | Provides a mechanism for authenticating TCP segments.                                                                                                                  | Enhances security by preventing unauthorized segment insertion.          |
| Multipath TCP (MPTCP)          | Allows multiple paths for a single TCP flow.                                                                                                                             | Improves redundancy and potentially increases throughput.               |
| TCP Fast Open (TFO)            | Allows data to be sent during the initial handshake process.                                                                                                             | Reduces the latency involved in opening a TCP connection.               |

This table includes a range of TCP options designed to address various network conditions, performance optimization needs, and security requirements, showcasing the protocol's adaptability and evolution over time.

## Maximum Segment Size
During connection establishment, the client and the server set the MSS option in their SYN packet.
MSS is calculated as follows:
MSS = Interface MTU - IP header size - TCP header size
    = 1500 - 20 - 20
    = 1460 bytes

### Path MTU Discovery
  - **Don't Fragment (DF) bit**: The source sets the DF bit in the IP header of outgoing packets. This instructs routers not to fragment these packets.
  - **Sending Initial Packets**: The source initially sends packets with MTU of its local interface.
  - **Fragmentation Needed and DF set**: If any of the packets encounter a router with a smaller MTU on the path to the destination, the router drops the packet and sends back an ICMP "Fragmentation Needed and DF set" message to the source. This message includes the MTU of that link indicating the largest packet size it can forward.
  - **Adjusting the MTU**: The source adjusts its packet size to match the reported MTU and retransmits the packet.
For IPV6, ICMPv6 "Packet too big" message is generated.

## Timestamps
The TCP timestamps option serves two primary purposes:
  - RTT (Round Trip Time)
  - PAWS (Protection against wrapped sequence numbers)
It consists of two fields, each 4 bytes long:
  - **TS Value (TSval)**: Contains the current value of the timestamp clock of the TCP sender.
  - **TS Echo Reply (TSecr)**: Used to echo the most recent Timestamp value received from the remote TCP endpoint.

### Enabling Timestamps:
  -  The use of the timestamps option is negotiated at the start of the TCP connection. Both TCP endpoints must indicate their support for the option in the SYN segments during the 3-way segment.
  -  Once enabled, all TCP segments sent during the connection should include the Timestamps options

### Round-Trip Time Measurement:
  - **Timestamping packets:** When a TCP segment is sent, the sender includes its current timestamp in the TSval field.
  - **Echoing Timestamps:** The receiver echoes this timestamp back to the sender in the TSecr field of its ACK packets.
  - **Calculating RTT:** By comparing the echoed timestamp with its current timestamp, the sender can accurately calculate RTT.

### Protection against Wrapped Sequence Numbers (PAWS):
  - **Sequence Number Wraparound:** With high-speed networks, and long duration connections, there's a theoritical risk that TCP sequence numbers might wrap around.
  - **Timestamps as a Solution:** The Timestamps option helps to distinguish between new and old segments that might have the same sequence numbers, by checking the timestamp values. A TCP segment is discarded if it has a sequence number that appears to be valid but carries an older timestamp than expected.
  - **Timestamp Validation:** Each TCP segment's timestamp is compared against the last received timestamp to ensure that no out-of-date segments are accepted.

