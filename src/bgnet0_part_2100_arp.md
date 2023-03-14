# ARP: The Address Resolution Protocol

As a networked computer, we have a problem.

We want to send data on the LAN to another computer on the same subnet.

Here's what we need to know in order to build an Ethernet frame:

* The data we want to send and its length
* Our source MAC address
* Their destination MAC address

Here's what we know:

* The data we want to send and its length
* Our source MAC address
* Our source IP address
* Their destination IP address

What's missing? Even though we know the other computer's IP address, _we
don't know their MAC address_. How can we build an Ethernet frame
without their MAC address? We can't. We must get it somehow.

> Again, for this section we're talking about sending on the LAN, the
> local Ethernet. Not over the Internet with IP. This is between two
> computers on the same Physical Layer network.

This section is all about ARP, the Address Resolution Protocol. This is
how one computer can map a different computer's IP address to that
computer's MAC address.

## Ethernet Broadcast Frames

We need some background first, though.

Recall that network hardware listens for Ethernet frames that are
addressed specifically to it. Ethernet frames for other MAC address
destinations are ignored.

> Side note: they are ignored unless the network card is placed into
> [_promiscuous mode_](https://en.wikipedia.org/wiki/Promiscuous_mode),
> in which case it receives **all** traffic on the LAN and forwards it
> to the CPU.

But there's a way to override: the _broadcast frame_. This is a frame
that has a destination MAC address of `ff:ff:ff:ff:ff:ff`. All devices
on the LAN will received that frame.

We're going to make use of this with ARP.

## ARP--The Address Resolution Protocol

So we have the destination's IP address, but not its MAC address. We
want its MAC address.

Here's what's going to happen:

1. The source computer will broadcast a specialized Ethernet frame that
   contains the destination IP address. This is the _ARP request_.

   (Remember the EtherType field from the previous exploration? ARP
   packets have EtherType 0x0806 to differentiate them from regular data
   Ethernet packets.)

2. All computers on the LAN receive the ARP request and examine it. But
   only the computer with the IP address specified in the ARP request
   will continue. The other computers discard the packet.

3. The destination computer with the specified IP address builds an _ARP
   response_. This Ethernet frame contains the destination computer's
   MAC address.

4. The destination computer sends the ARP response back to the source
   computer.

5. The source computer receives the ARP response, and now it knows the
   destination computer's MAC address.

And it's game-on! Now that we know the MAC address, we can send with
impunity.

## ARP Caching

Since it's a pain to ask a computer for its MAC address every time we
want to send it something, we'll _cache_ the result for a while.

Then, when we want to send to a particular IP on the LAN, we can look in
the _ARP cache_ and see if the IP/Ethernet pair is already there. If so,
no need to send out an ARP request--we can just transmit the data right
away.

The entries in the cache will timeout and be removed after a certain
amount of time. There's no standard time to expiration, but I've seen
numbers from 60 seconds to 4 hours.

Entries could go stale if the MAC address changes for a given IP
address. Then the cache entry would be out of date. The easiest way for
that to happen would be if someone closed their laptop and left the
network (taking their MAC address with them), and then another person
with a different laptop (and different MAC address) showed up and was
assigned the same IP address. If that happened, computers with the
stale entry would send the frames for that IP to the wrong (old) MAC
address.

## ARP Structure

The ARP data goes in the payload portion of the Ethernet frame. It's
fixed length. As mentioned before, it's identified by setting the
EtherType/packet length field to 0x0806.

In the payload structure below, when it says "Hardware" it means the
Link Layer (e.g. Ethernet in this example) and when it says "Protocol"
it means Network Layer (e.g. IP in this example). It uses those
generalized names for the fields since there's no requirement that ARP
use Ethernet or IP--it can work with other protocols, too.

The payload structure, with a total fixed length of 28 octets:

* 2 octets: Hardware Type (Ethernet is `0x0001`)
* 2 octets: Protocol Type (IPv4 is `0x8000`)
* 1 octet: Hardware address length in octets (Ethernet is `0x06`)
* 1 octet: Protocol address length in octets (IPv4 is `0x04`)
* 2 octets: Operation (`0x01` for request, `0x02` for reply)
* 6 octets: Sender hardware address (Sender MAC address)
* 4 octets: Sender protocol address (Sender IP address)
* 6 octets: Target hardware address (Target MAC address)
* 4 octets: Target protocol address (Target IP address)

## ARP Request/Response

This gets a little confusing, because the "Sender" fields are always set
up from the point of view of the computer doing the transmitting, not
from the point of view of who is the requester.

So we're going to declare that Computer 1 is the sending the ARP
request, and Computer 2 is going to respond to it.

In an ARP request from Computer 1 ("If you have this IP, what's your
MAC?"), the following fields are set up (in addition to the rest of the
boilerplate ARP request fields mentioned above):

* **Sender hardware address**: Computer 1's MAC address
* **Sender protocol address**: Computer 1's IP address
* **Target hardware address**: unused
* **Target protocol address**: the IP address Computer 1 is curious
  about

In an ARP response from Computer 2 ("I have that IP, and this is my
MAC."), the following fields are set up:

* **Sender hardware address**: Computer 2's MAC address
* **Sender protocol address**: Computer 2's IP address
* **Target hardware address**: Computer 1's MAC address
* **Target protocol address**: Computer 1's IP address

When Computer 1 receives the ARP reply that names it as the target, it
can look in the Sender fields and get the MAC address and its
corresponding IP address.

After that, Computer 1 is able to send Ethernet traffic to Computer 2's
now-known MAC address.

And that is how MAC addresses are discovered for a particular IP
address!

## Other ARP Features

### ARP Announcements

It's not uncommon for computers that have just come online to announce
their ARP information unsolicited. This gives everyone else a chance to
add the data to their caches, and it overwrites potentially stale data
in those caches.

### ARP Probe

A computer can send out a specially-constructed ARP request that
basically asks, "Is anyone else using this IP address?"

Typically it asks using its own IP address; if it gets a response, it
knows it has an IP conflict.

## IPv6 and ARP

IPv6 has its own version of ARP called
[NDP](https://en.wikipedia.org/wiki/Neighbor_Discovery_Protocol) (the
Neighbor Discovery Protocol).

The eagle-eyed among you might have noticed that ARP only supports
protocol addresses (e.g. IP addresses) of up to 4 bytes, and IPv6
addresses are 16 bytes.

NDP addresses this issue and more, defining a number of
[ICMPv6](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol_for_IPv6)
(Internet Message Control Protocol for IPv6) that can be used to take
the place of ARP, among other things.

## Reflect

* Describe the problem ARP is solving.

* Why do entries in ARP caches have to expire?

* Why can't IPv6 use ARP?
