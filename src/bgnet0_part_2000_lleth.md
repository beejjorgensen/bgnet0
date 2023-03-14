# The Link Layer and Ethernet

We're getting down to the guts of the thing: The Link Layer.

<!-- CAPTION: Internet Layered Network Model -->
|Layer|Responsibility|Example Protocols|
|:-:|-|-|
|Application|Structured application data|HTTP, FTP, TFTP, Telnet, SSH, SMTP, POP, IMAP|
|Transport|Data Integrity, packet splitting and reassembly|TCP, UDP|
|Internet|Routing|IP, IPv6, ICMP|
|Link|Physical, signals on wires|Ethernet, PPP, token ring|

The link layer is where all the action happens, where bytes turn into
electricity.

This is where Ethernet lives, as we'll soon see.

## A Quick Note on Octets

We've been using "byte" as a word meaning 8 bits, but now it's time to
get more specific.

Historically, see, a byte didn't have to be 8 bits. (Famously, the C
programming language doesn't specify how many bits are in a byte,
exactly.) And there's nothing preventing computer designers from just
making things up. It only so happens that basically every modern
computer uses 8 bits for a byte.

In order to be more precise, people sometimes use the word _octet_ to
represent 8 bits. It's defined to be exactly 8 bits, period.

When someone casually says (or writes) "byte", they probably mean 8
bits. When someone says "octet" they most definitely mean exactly 8
bits, end of story.

## Frames versus Packets

When we get to the Link Layer, we've got a bit more terminology to get
used to. Data sent out over Ethernet is a packet, but within that packet
is a _frame_. It's like a sub-packet.

Conversationally, people use Ethernet "frame" and "packet"
interchangeably. But as we'll see later, there is a differentiation in
the full ISO OSI layered network model.

In the simplified Internet layered network model, the differentiation
is not made, thus leading to the confusing terminology.

More on this captivating tale later in this exploration.

## MAC Addresses

Every network interface device has a MAC (Media Access Control) address.
This is an address that's unique on the LAN (local area network) that's
used for sending and receiving data.

An Ethernet MAC address comes in the form of 6 hex bytes (12 hex digits)
separated by colons (or hyphens or periods). For example, these are all
ways you might see a MAC address represented:

```
ac:d1:b8:df:20:85
ac-d1-b8-df-20-85
acd1.b8df.2085
```

MAC address must be unique on the LAN. The numbers are assigned at
manufacturer and are not typically changed by the end user. (You'd only
want to change them if you happened to get unlucky and buy two network
cards that happened to have been assigned the same MAC address.)

The first three bytes of an Ethernet MAC address are called the _OUI_
(Organizationally Unique Identifier) that is assigned to a manufacturer.
This leaves each manufacturer three bytes to uniquely represent the
cards they make. (That's 16,777,216 possible unique combinations. If a
manufacturer runs out, they can always get another OUI--there are 16
million of those available, too!)

> Funny Internet rumor: there was once a manufacturer of knockoffs of a
> network card called the NE2000, itself already known as a "bargain"
> network card. The knockoff manufacturer took the shortcut of burning
> the same MAC address into every card they made. This was discovered
> when a company bought a large number of them and found that only one
> computer would work at a time. Of course, in a home LAN where someone
> was only likely to have one of these cards, it wouldn't be a
> problem--which is what the manufacturer was banking on. To add insult
> (or perhaps injury) to injury, there was no way to change the MAC
> address in the knockoff cards. The company was forced to discard them
> all.
>
> Except one, presumably.

## We're All In The Same Room (Typically)

Lots of modern link layer protocols that you'll be directly exposed to
operate on the idea that they're all broadcasting into a shared medium
and listening for traffic that's addressed to them.

It's like being in a room full of talkative people and someone shouts
your name--you get the data that's addressed to you and everyone else
ignores it.

This works in real life and in protocols like Ethernet. When you
transmit a packet on an Ethernet network, everyone else on the same
Ethernet LAN can see that traffic. It's just that their network cards
ignore the traffic unless it's specifically addressed to them.

> Asterisks: there are a couple handwavey things in that paragraph.
>
> One is that in a modern wired Ethernet, a device called a _switch_
> typically prevents packets from going out to nodes that they're not
> supposed to. So the network isn't really as loud as the crowded-room
> analogy suggests. Back before switches, we used things called `hubs`,
> which didn't have the brains to discriminate between destinations.
> They'd broadcast all Ethernet 

Not every link layer protocol works this way, however. The common goal
of all of them is that we're going to send and receive data at the wire
level.

## Multiple Access Method

Ready for some more backstory?

If everyone on the same Ethernet is in the same room yelling at other
people, how does that actually work on the wire? How do multiple
entities access a shared medium in a way that they don't conflict?

> When we're talking "medium" here, we mean wires (if you've plugged
> your computer into the network) or radio (if you're on WiFi).

The method particular link layer protocols use to allow multiple
entities access the shared medium is called the _multiple access
method_, or _channel access method_.

There are [a number of ways of doing
this](https://en.wikipedia.org/wiki/Channel_access_method). On the same
medium:

* You could transmit packets on different frequencies.
* You could send packets at different times, like timesharing.
* You could use spread spectrum or frequency hopping.
* You could split the network into different "cells".
* You could add another wire to allow traffic to flow both directions at
  once.

Let's again use Ethernet as an example. Ethernet is most like the
"timesharing" mode, above.

But that still leaves a lot of options open for exactly how we do
_that_.

Wired Ethernet uses something called
[CSMA/CD](https://en.wikipedia.org/wiki/Carrier-sense_multiple_access_with_collision_detection)
(Carrier-Sense Multiple Access with Collision Detection). Easy for you
to say.

This method works like this:

1. The Ethernet card waits for quiet in the room--when no other network
   card is transmitting. (This is the "CSMA" part of CSMA/CD.)
2. It starts sending.
3. It also listens while it's sending.
4. If it receives the same thing that it sent, all is well.

   If it doesn't receive the same thing it sent, it means that another
   network device also started transmitting at the same time. This is a
   collision detection, the "CD" part of CSMA/CD. 5. 

   To resolve the situation, the network card transmits a special signal
   called the "jam signal" to alert other cards on the network that a
   collision has occurred and they should stop transmitting. The network
   card then waits a small, partly random amount of time, and then goes
   back to step 1 to retry the transmission.

WiFi (wireless) Ethernet uses something similar, except it's
[CSMA/CA](https://en.wikipedia.org/wiki/Carrier-sense_multiple_access_with_collision_avoidance)
(Carrier-Sense Multiple Access with Collision Avoidance). Also easy for
you to say.

This method works like this:

1. The Ethernet card waits for quiet in the room--when no other network
   card is transmitting. (This is the "CSMA" part of CSMA/CA.)
  
2. If the channel isn't quiet, the network card waits a small, random
   amount of time, then goes back to step 1 to retry.

There are a few more details omitted there, but that's the gist of it.

## Ethernet

Remember with the layered network model how each layer _encapsulates_
the previous layer's data into its own header?

For example, HTTP data (Application Layer) gets wrapped up in a TCP
(Transport Layer) header. And then all of that gets wrapped up in an IP
(Network Layer) header. And then **all** of that gets wrapped up in an
Ethernet (Link Layer) frame.

And recall that each protocol had its own header structure that helped
it perform its job.

Ethernet is no different. It's going to encapsulate the data from the
layer above it.

Now, I want to get a little picky about terminology. The whole chunk of
data that's transmitted is the Ethernet _packet_. But within it is the
Ethernet _frame_. As we'll see later, these correspond to two layers of
the ISO OSI layered network model (that have been condensed into a
single "Link layer" in the Internet layered network model).

Though I've written the frame "inside" the packet here, note that they
are all transmitted as a single bitstream.

* **The Packet:**
  * 7 octets: Preamble (in hex: `AA AA AA AA AA AA AA`)
  * 1 octet: Start frame delimiter (in hex: `AB`)
  * **The Frame:**
    * 6 octets: Destination MAC address
    * 6 octets: Source MAC address
    * 4 octets: ["Dot1q" tag](https://en.wikipedia.org/wiki/IEEE_802.1Q)
      for [virtual LAN](https://en.wikipedia.org/wiki/VLAN)
      differentiation.
    * 2 octets: Payload Length/Ethertype (see below)
    * 46-1500 octets: Payload
    * 4 octets: [CRC-32 checksum](https://en.wikipedia.org/wiki/Cyclic_redundancy_check#CRC-32_algorithm)
  * End of frame marker, loss of carrier signal
  * Interpacket gap, enough time to transmit 12 octets

The Payload Length/[EtherType](https://en.wikipedia.org/wiki/EtherType)
field is used for the payload length in normal usage. But other values
can be put there that indicate an alternate payload structure.

The largest payload that can be transmitted is 1500 octets. This is
known as the MTU (Maximum Transmission Unit) of the network. Data that
is larger must be fragmented down to this size.

> Ethernet hardware can use this 1500 number to differentiate the
> Payload Length/EtherType header field. If it's 1500 octets or fewer,
> it's a length. Otherwise it's an EtherType value.

After the frame, there's an end-of-frame marker. This is indicated by
loss of carrier signal on the wire, or by some explicit transmission in
some versions of Ethernet.

Lastly, there's a time gap between Ethernet packets which corresponds to
the amount of time it would take to transmit 12 octets.

### Two Layers of Ethernet?

If you recall, our simplified layer model is actually a crammed-down
version of the full ISO OSI 7-Layer model:

<!-- CAPTION: ISO OSI Network Layer Model -->
|ISO OSI Layer|Responsibility|Example Protocols|
|:-:|-|-|
|Application|Structured application data|HTTP, FTP, TFTP, Telnet, SMTP, POP, IMAP|
|Presentation|Encoding translation, encryption, compression|MIME, SSL/TLS, XDR|
|Session|Suspending, terminating, restarting sessions between computers|Sockets, TCP|
|Transport|Data integrity, packet splitting and reassembly|TCP, UDP|
|Network|Routing|IP IPv6, ICMP|
|Data link|Encapsulation into frames|Ethernet, PPP, SLIP|
|Physical|Physical, signals on wires|Ethernet physical layer, DSL, ISDN|

What we call the Internet "Link Layer" is the "Data Link" layer _and_
the "Physical" layer.

The Data Link layer of Ethernet is the _frame_. It's a subset of the
entire _packet_ (outlined above) which is defined at the Physical Layer.

Another way to consider this is that the Data Link Layer is busy
thinking about logical entities like who has what MAC address and what
the payload checksum is. And that that Physical Layer is all about
figuring out which patterns of signals to send that correspond to the
start and end of the packet and frame, when to lower the carrier signal,
and how long to delay between transmissions.

## Reflect

* What's your MAC address on your computer? Do an Internet search to
  find how to look it up.

* What's the deal with _frames_ versus _packets_ in Ethernet? Where in
  the ISO OSI network stack do they live?

* What's the difference between a byte and an octet?

* What's the main difference between CSMA/CD and CSMA/CA?

