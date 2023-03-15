# User Datagram Protocol (UDP)

If you like to keep things simple and are a positive thinker, UDP is for
you. It's the near-ultimate in lightweight data transfer over the
Internet.

You fire UDP packets off and hope they arrive. Maybe they do, or maybe
someone put a backhoe through a fiber optic cable, or there were cosmic
rays, or a router got too congested or irate and just dropped it.
Unceremoniously.

It's living on the Internet data edge! Virtually all the pleasant
and reliable guarantees of TCP--gone!

## Goals of UDP

* Provide a way to send error-free data from one computer to another.

That's about it.

The following are **not** goals of UDP:

* Provide data in order
* Provide data without loss
* Provide data without duplicates

If those are needed, TCP is a better choice. UDP offers zero protection
against lost or misordered data. The only guarantee is that _if_ the
data arrives, it will be correct.

But what it does give you is really low overhead and quick response
times. It doesn't have any of the packet reassembly or flow control or
ACK packets or any of the stuff TCP does. As a consequence, the header
is much smaller.

## Location in the Network Stack

Recall the layers of the Network Stack:

<!-- CAPTION: Internet Layered Network Model -->
|Layer|Responsibility|Example Protocols|
|:-:|-|-|
|Application|Structured application data|HTTP, FTP, TFTP, Telnet, SSH, SMTP, POP, IMAP|
|Transport|Data Integrity, packet splitting and reassembly|TCP, UDP|
|Internet|Routing|IP, IPv6, ICMP|
|Link|Physical, signals on wires|Ethernet, PPP, token ring|

You can see UDP in there at the Transport Layer. IP below it is
responsible for routing. And the application layer above it takes
advantage of all the features UDP has to offer. Which isn't a lot.

## UDP Ports

UDP uses ports, similar to TCP. In fact, you can have a TCP program
using the same port number as a different UDP program.

IP uses IP addresses to identify hosts.

But once on that host, the port number is what the OS uses to deliver
the data to the correct process.

By analogy, the IP address is like a street address, and the port number
is like an apartment number at that street address.

## UDP Overview

UDP is _connectionless_. You know how TCP takes the packet-switched
network and makes it look like it's a reliable connection between two
computers? UDP doesn't do that.

You send a UDP datagram to an IP address and port. IP routes it there
and the receiving computer sends it to the program that's bound to that
port.

There's no connection. It's all on an individual packet basis.

When the packet arrives, the receiver can tell which IP and port it came
from. This way the receiver can send a response.

## Data Integrity

There are a lot of things that can go wrong. Data can arrive out of
order. It can be corrupted. It might be duplicated. Or maybe it doesn't
arrive at all.

UDP barely has any mechanisms to handle all these contingencies.

In fact, all it does is error detection.

### Error Detection

Before the sender sends out a segment, a _checksum_ is computed for that
packet.

The checksum works exactly the same way as with TCP, except the UDP
header is used. Compared to the TCP header, the UDP header is dead
simple:

``` {.default}
 0      7 8     15 16    23 24    31  
+--------+--------+--------+--------+ 
|     Source      |   Destination   | 
|      Port       |      Port       | 
+--------+--------+--------+--------+ 
|                 |                 | 
|     Length      |    Checksum     | 
+--------+--------+--------+--------+ 
|                                     
|          data octets ...            
+---------------- ...                 
```

When the receiver gets the packet, it computes its own checksum of that
packet.

If the two checksums match, the data is assumed to be error-free. If
they differ, the data is discarded.

That's it. The receiver never even knows that there was some data
aimed at it. It just vanishes into the ether.

The checksum is a 16-bit number that is the result of piping all the UDP
header and payload data and the IP addresses involved into a function
that digests them down.

This works the same way as the TCP checksum. (Jon Postel wrote the first
RFCs for both TCP and UDP so it's no surprise they use the same
algorithm.)

The details of how the checksum works are in this week's project. Just
substitute the UDP header for the TCP header.

## Maximum Payload Without Fragmentation

It's a little ahead of schedule, but lower layers can decide to split a
UDP packet up into smaller packets. Maybe there's a part of the internet
the UDP packet has to pass through that can only handle sending data of
a certain size.

We call this "fragmentation", when a UDP packet is split among multiple
IP packets.

The maximum size packet that can be sent on any particular wire is
called its MTU (maximum transmission unit). The smallest possible MTU on
the Internet (IPv4) is 576 bytes. The biggest IP header is 60 bytes. And
the UDP header is 8 bytes. So that leaves 576-60-8 = 508 bytes of
payload that you can guarantee won't be fragmented. Since the IP header
is sometimes smaller than 60 bytes, lots of source say 512 bytes is the
limit.

Is fragmentation bad? Some routers might drop fragmented UDP packets. So
staying under the minimum MTU is often a good idea with UDP.

## What's the Use?

If UDP can drop packets all over the place, why ever use it?

Well, the performance gain is notable, so that's a draw.

There are basically two circumstances you would use UDP:

1. If you don't care if you lose a few packets. If you're transmitting
   voice or video or even game frame information, it might be OK to drop
   a few packets. The stream will just glitch out for a moment and then
   continue when the next packets arrive.
   
   This is the most common use. Multiplayer high-framerate games use
   this from frame-by-frame updates, and also use TCP for lower
   bandwidth needs like chat messages and player inventory changes.

2. If you can't have packet loss, you can implement another protocol on
   top of UDP. The TFTP (Trivial File Transfer Protocol) does this. It
   puts a sequence number in each packet, and waits for the other side
   to respond with a TFTP ACK packet before sending another. It's not
   fast because the sender has to wait for an ACK before sending the
   next packet, but it's really simple to implement.

   This is a rarer use. TFTP is used by diskless computers that don't
   have an OS installed. They transfer the OS over the network on boot,
   and need a small built-in network stack to make that happen. It's a
   lot easier to implement an Ethernet/IP/UDP stack than an
   Ethernet/IP/TCP stack.

## UDP (Datagram) Sockets

With UDP sockets, there are some differences from TCP:

* You no longer call `listen()`, `connect()`, `accept()`, `send()`, or
  `recv()` because there's no "connection".
* You call `sendto()` to send UDP data.
* You call `recvfrom()` to receive UDP data.

### Server Procedure

The general procedure for a server is to make a new socket of type
`SOCK_DGRAM`, which is a datagram/UDP socket. (We have been using the
default of `SOCK_STREAM` which is a TCP socket.)

Then the server calls `bind()` to bind to a port. This port is where the
client will send packets.

After that, the server can loop receiving data and sending responses.

When it receives data, `recvfrom()` will return the host and port the
data was sent from. This can be used to reply, sending data back to the
sender.

Here's an example server:

``` {.py}
# UDP Server

import sys
import socket

# Parse command line
try:
    port = int(sys.argv[1])
except:
    print("usage: udpserver.py port", file=sys.stderr)
    sys.exit(1)

# Make new UDP (datagram) socket
s = socket.socket(type=socket.SOCK_DGRAM)

# Bind to a port
s.bind(("", port))

# Loop receiving data
while True:
    # Get data
    data, sender = s.recvfrom(4096)
    print(f"Got data from {sender[0]}:{sender[1]}: \"{data.decode()}\"")

    # Send a reply back to the original sender
    s.sendto(f"Got your {len(data)} byte(s) of data!".encode(), sender)
```

### Client Procedure

It's about the same as the server procedure, except it's not going to
`bind()` to a specific port. It'll let the OS choose bind port for it
the first time it calls `sendto()`.

Remember: UDP is unreliable, so there's a chance the data might not
arrive! If it doesn't, just try again. (But it will almost certainly
arrive running on localhost.)

Example client that can communicate with the above server:

``` {.py}
# UDP Client

import socket
import sys

# Parse command line
try:
    server = sys.argv[1]
    port = int(sys.argv[2])
    message = sys.argv[3]
except:
    print("usage: udpclient.py server port message", file=sys.stderr)
    sys.exit(1)

# Make new UDP (datagram) socket
s = socket.socket(type=socket.SOCK_DGRAM)

# Send data to the server
print("Sending message...")
s.sendto(message.encode(), (server, port))

# Wait for a reply
data, sender = s.recvfrom(4096)
print(f"Got reply: \"{data.decode()}\"")

s.close()
```

## Reflect

* What does TCP provide that UDP does not in terms of delivery
  guarantees?

* Why do people recommend keeping UDP packets small?

* Why is the UDP header so much smaller than the TCP header?

* `sendto()` requires you specify a destination IP and port. Why does
  the TCP-oriented `send()` function not require those arguments?

* Why would people use UDP over TCP if it's relatively unreliable?
