# Transmission Control Protocol (TCP)

When someone puts a backhoe through a fiber optic cable, packets might
be lost. (Entire countries have been brought offline by having a boat
anchor dragged through an undersea network cable!) Software errors and
computer crashes and router malfunctions can all cause problems.

But we don't want to have to think about that. We want an entity to deal
with all that and then let us know when the data is complete and intact
and in order.

TCP is that entity. It worries about lost packets so we don't have to.
And when it is sure it has all the correct data, _then_ it gives it to
us.

We'll look at:

* The overall goals of TCP
* Where it fits in the network stack
* A refresher on TCP ports
* How TCP makes, uses, and closes connections
* Data integrity mechanisms
  * Maintaining packet order
  * Detecting errors
* Flow control--how a receiver keeps from getting overwhelmed
* Congestion Control--how senders avoid overloading the Internet

TCP is a very complex topic and we're only skimming the highlights here.
If you want to learn more, the go-to book is _TCP/IP Illustrated Volume
1_ by the late, great W. Richard Stevens.

## Goals of TCP

* Provide reliable communication
* Simulate a circuit-like connection on a packet-switched network
* Provide flow control
* Provide congestion control
* Support out-of-band data

## Location in the Network Stack

Recall the layers of the Network Stack:

<!-- CAPTION: Internet Layered Network Model -->
|Layer|Responsibility|Example Protocols|
|:-:|-|-|
|Application|Structured application data|HTTP, FTP, TFTP, Telnet, SSH, SMTP, POP, IMAP|
|Transport|Data Integrity, packet splitting and reassembly|TCP, UDP|
|Internet|Routing|IP, IPv6, ICMP|
|Link|Physical, signals on wires|Ethernet, PPP, token ring|

You can see TCP in there at the Transport Layer. IP below it is
responsible for routing. And the application layer above it takes
advantage of all the features TCP has to offer.

That's why when we wrote our HTTP client and server, we didn't have to
worry about data integrity at all. We used TCP so that took care of it
for us!

## TCP Ports

Recall that when we used TCP we had to specify port numbers to connect
to. And even our clients were automatically assigned local ports by the
operating system (if we didn't `bind()` them ourselves).

IP uses IP addresses to identify hosts.

But once on that host, the port number is what the OS uses to deliver
the data to the correct process.

By analogy, the IP address is like a street address, and the port number
is like an apartment number at that street address.

## TCP Overview

There are three main things TCP does during a connection:

1. Make the connection
2. Transmit data
3. Close the connection

Each of these involves the sending of special non-user-data packets back
and forth between the client and server. We'll look at special packet
types SYN, SYN-ACK, ACK, and FIN.

### Making the Connection

This involves the famous "three-way handshake". Since any packets can be
lost during transmission, TCP takes extra care to make sure both sides
of the connection are ready for data before proceeding.

1. The client sends a SYN (synchronize) packet to the server.
2. The server replies with a SYN-ACK (synchronize acknowledge) packet
   back to the client.
3. The client replies with an ACK (acknowledge) packet back to the
   server.

If there is no reply in a reasonable time to any one of these steps, the
packet is resent.

### Transmitting Data

TCP takes a stream of data and splits it into chunks. Each chunk gets a
TCP header attached to it, and is then sent on to the IP layer for
delivery. The header and the chunk together are called a TCP _segment_.

(We'll use "TCP packet" and "TCP segment" interchangeably, but segment
is more correct.)

When TCP sends a segment, it expects the recipient of that data to
respond with an acknowledgment, hereafter known as an ACK. If TCP
doesn't get the ACK, it's going to assume something has gone wrong and
it needs to resend that segment.

Segments are numbered so even if they arrive out of order, TCP can put
them back in the proper sequence.

### Closing the Connection

When a side wants to close the connection, it sends a FIN (finis
[_sic_]) packet. The remote side will typically reply with an ACK and a
FIN of its own. The local side would then complete the hangup with
another ACK.

In some OSes, if a host closes a connection with unread data, it sends
back a RST (reset) to indicate that condition. Socket programs might
print the message "Connection reset by peer" when this happens.

## Data Integrity

There are a lot of things that can go wrong. Data can arrive out of
order. It can be corrupted. It might be duplicated. Or maybe it doesn't
arrive at all.

TCP has mechanisms to handle all these contingencies.

### Packet Ordering

The sender places an ever-increasing sequence number on each segment.
"Here's segment 3490. Here's segment 3491."

The receiver replies to the sender with an ACK packet containing that
sequence number. "I got segment 3490. I got segment 3491."

If two segments arrive out of order, TCP can put them back in order by
sorting them by sequence number.

> Analogy time! If you had a stack of papers and threw them in the air,
> how could you possibly know their original order? Well, if you
> numbered all the pages correctly, you just have to sort them. That's
> the role the sequence number plays in TCP.

If a duplicate segment arrives, TCP knows it's already seen that
sequence number, so it can safely discard the duplicate.

If a segment is missing, TCP can ask for a retransmission. It does this
by repeatedly ACKing the previous segment. The sender will retransmit
the next one.

Alternately, if the sender doesn't get an ACK for a particular segment
for some time, it might retransmit that segment on its own, thinking the
segment might have been lost. This retransmission gets more and more
pessimistic with each timeout; the server doubles the timeout each time
it happens.

Lastly, sequence numbers are initialized to random values during the
three-way handshake as the connection is being made.

### Error Detection

Before the sender sends out a segment, a _checksum_ is computed for that
segment.

When the receiver gets the segment, it computes its own checksum of that
segment.

If the two checksums match, the data is assumed to be error-free. If
they differ, the data is discarded and the sender must timeout and
retransmit.

The checksum is a 16-bit number that is the result of piping all the TCP
header and payload data and the IP addresses involved into a function
that digests them down.

The details are in this week's project.

## Flow Control

_Flow Control_ is the mechanism by which two communicating devices alert
one another that data needs to be sent more slowly. You can't pour 1000
Mbs (megabits per second) at a device that can only handle 100 Mbs. The
device needs a way to alert the sender to slow it down.

> Analogy time: you do this on the phone when you tell the other party
> "You're talking too fast for me to understand! Slow down!"

The most simple way to do this (and this is not what TCP does) is for
the sender to send the data, then wait for the receiver to send back an
ACK packet with that sequence number. Then send another data packet.
This way the receiver can delay the ACK if it needs the sender to slow
down.

But this is a slow back and forth, and the network is usually reliable
enough for the sender to push out multiple segments without waiting for
a response.

However, if we do this, we risk the sender sending data more quickly
than the receiver can handle it!

In TCP, this is solved with something called a _sliding window_. This
goes in the "window" field of the TCP header in the receiver's ACK
packet.

In that field, the data receiver can specify how much more data (in
bytes) it is willing to receive. The sender must not send more than this
without getting an ACK from the receiver. And the ACK it gets
will contain new window information.

Using the mechanism, the receiver can get the sender "once you've sent X
bytes, you have to wait for an ACK telling you how many more you can
send".

It's important to note that this is a byte count, not a segment count.
The sender is free to send multiple segments without receiving an ACK as
long as the total bytes doesn't exceed the receiver's advertised window
size.

## Congestion Control

Flow control operates between two computers, but there's a greater issue
of the Internet as a whole. If a router becomes overwhelmed, it might
start dropping packets which causes the sender to begin retransmitting,
which does nothing to alleviate the problem. And it's not even on flow
control's radar since the packets aren't reaching the receiver.

This happened in 1986 when
[NSFNET](https://en.wikipedia.org/wiki/National_Science_Foundation_Network)
(basically the pre-commercial Internet, may it rest in peace) was
overwhelmed by insistent senders who didn't know when to quit with the
retransmissions. Throughput dropped to 0.1% of normal. It wasn't great.

To fix this, a number of mechanisms were implemented by TCP to estimate
and eliminate network congestion. Note that these are in addition to the
Flow Control window advertised by a receiver. The sender must obey the
Flow Control limit **and** the computed network congestion limit,
whichever is lower. It cannot have more unacknowledged segments out on
the network than this limit. If it does, it has to stop sending and wait
for some ACKs to come in.

Another way to think about this is that when a sender puts out a new TCP
segment, that adds to network congestion. When it receives an ACK, that
indicates that the segment has been removed from the network, decreasing
congestion.

In order to alleviate the problems that hit NFSNET, two big algorithms
were added: Slow Start and Congestion Avoidance.

**NOTE:** This is a simplified view of these two algorithms. For full
details on the complex interplay between them and even more on
congestion avoidance, see [_TCP Congestion Control_ (RFC
5681)](https://www.rfc-editor.org/rfc/rfc5681.html).

### Slow Start

When the connection first comes up, the sender has no way of knowing how
congested the network is. This first phase is all about getting a rough
guess sorted out.

And so it's going to start conservatively, assuming there's a high
congestion level. (If there is already a high congestion level,
liberally flooding it with data wouldn't be helpful.)

It starts by allowing itself an initial _congestion window_ which is
how many unACKed bytes (and segments, but let's just think of bytes for
now) it is allowed to have outstanding. 

As the ACKs come in, the size of the congestion window increases by the
number of acknowledged bytes. So, loosely, after one segment gets ACKed,
two can be sent out. If those two are successfully ACKed, four can be
sent out.

So it starts with a very limited number of unACKed segments allowed to
be outstanding, but grows very rapidly.

Eventually one of those ACKs is lost and that's when Slow Start decides
to slow way down. It cuts the congestion window size by half and then
TCP switches to the Congestion Avoidance algorithm.

### Congestion Avoidance

This algorithm is similar to Slow Start, but it makes much more
controlled moves. No more of this exponential growth stuff.

Each time a congestion window's-worth of data is successfully
transmitted, it pushes a little harder by adding another segment's worth
of bytes to the window. This allows it to have another unACKed segment
out on the network. If that works fine, it allows itself one more. And
so on.

So it's pushing the limits of what can be successfully sent without
congesting the network, but it's pushing slowly. We call this
_additive-increase_ and it's generally linear (compared to Slow Start
which was generally exponential).

Eventually, though, it pushes too far, and has to retransmit a packet.
At this point, the congestion window is set to a small size and the
algorithm drops back to Slow Start.

## Reflect

* Name a few protocols that rely on TCP for data integrity.

* Why is there a three-way handshake to set up a connection? Why not
  just start transmitting?

* How does a checksum protect against data corruption?

* What's the main difference in the goals of Flow Control and Congestion
  Control?

* Reflect on the reasons for switching between Slow Start and Congestion
  Avoidance. What advantages does each have in different phases of
  congestion detection?

* What is the purpose of Flow Control?

