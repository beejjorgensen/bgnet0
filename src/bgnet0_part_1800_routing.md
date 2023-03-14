# IP Routing

This exploration is all about routing over IP. This task is handled by
computers called _routers_ that know how to direct traffic down
different lines that are connected to them.

As we'll see, every computer attached to the Internet is actually a
router, but they all rely on other routers to get the packets to their
destinations.

We're going to talk about two big ideas:

* **Routing Protocols**: how routers learn the "map" of the network.

* **Routing Algorithm**: how routers decide which direction to send
  packets after they've learned the map.

Let's get into it!

## Routing Protocols

Routing protocols in general have this goal: give routers enough
information to make routing decisions.

That is, when a router receives a packet on one interface, how does it
decide which interface to forward it to? How does it know which route
leads to the packet's eventual destination?

### Interior Gateway Protocols

When it comes to routing packets over the Internet, we'll take a higher
view. Let's look at the Internet as a bunch of clumps of networks that
are more loosely connected. "Clump" is not an official industry term,
for the record. The official term is _autonomous system_ (AS), which
Wikipedia defines as:

> [...] a collection of connected Internet Protocol (IP) routing
> prefixes under the control of one or more network operators on behalf
> of a single administrative entity or domain, that presents a common
> and clearly defined routing policy to the Internet.

But let's think of it as a clump for now.

For example, OSU has a clump of network. It has its own internal routers
and subnets that are "on the Internet". A [FAANG
company](https://en.wikipedia.org/wiki/Big_Tech#FAANG) might have
another clump of Internet.

Within these clumps of Internet, an _interior gateway protocol_ is used
for routing. It's a routing algorithm that is optimized for smaller
networks with a small number of subnets.

Here are some examples of interior gateway protocols

<!-- CAPTION: Interior Gateway Protocols -->
|Abbreviation|Name|Notes|
|-|-|-|
|OSPF|Open Shortest Path First|Most commonly used, IP layer|
|IS-IS|Intermediate System to Intermediate System|Link layer|
|EIGRP|Enhanced Interior Gateway Routing Protocol|Cisco semi-proprietary|
|RIP|Routing Information Protocol|Old, rarely used|

But of course you need to be able to communicate between these clumps of
Internet--the whole Internet is connected after all. It would be a
bummer if you couldn't use Google's servers from Oregon State. But
clearly Google's servers don't have a map of OSU's network... so how do
they know how to route traffic?

### Exterior Gateway Protocols

Having every router have a complete map of the Internet isn't practical.
Works fine on smaller clumps, but not on the whole thing.

So we change it up for communicate between these clumps and use an
_exterior gateway protocol_ for routing between the clumps.

Remember the official term for "clump"? Autonomous Systems. Each
autonomous system on the Internet is assigned an _autonomous system
number_ (ASN) that is used by the _border gateway protocol_ (BGP) to
help determine where to route packets.

<!-- CAPTION: Exterior Gateway Protocols -->
|Abbreviation|Name|Notes|
|-|-|-|
|BGP|Border Gateway Protocol|Used everywhere|
|EGP|Exterior Gateway Protocol|Obsolete|

BGP can work in two different modes: internal BGP and external BGP. In
internal mode, it acts as an interior gateway protocol, while external
mode acts as an external gateway protocol.

Here's a [great video from _Eye on
Tech_](https://www.youtube.com/watch?v=A1KXPpqlNZ4) that concisely
covers it. I highly recommend spending the two minutes watching this to
tie it all together:

<iframe width="560" height="315" src="https://www.youtube.com/embed/A1KXPpqlNZ4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Routing tables

A routing table is a definition of where to forward packets to get them
farther along to their destination. Or, if the router is on the
destination LAN, the routing table directs the router to send the
traffic out locally at the physical layer (e.g. Ethernet).

The goal of the routing protocols we talked about above is to help
routers across the Internet build their routing tables.

All computers connected to the Internet--routers and otherwise--have
routing tables. Even on your laptop, the OS has to know what to do with
different destination addresses. What if the destination is localhost?
What if it's another computer on the same subnet? What if it's something
else?

Looking at the typical laptop as an example, the OS keeps localhost
traffic on the _loopback device_ and it typically doesn't go out on the
network at all. From our programmer standpoint, it looks like it's doing
network stuff, but the OS knows to route `127.0.0.1` traffic internally.

But what if you ping another computer on the same LAN as you? In that
case, the OS checks the routing table, determines it's on the same LAN,
and sends it out over Ethernet to the destination.

But what if you ping another computer on a completely different LAN than
you? When that happens, the OS can't just send out an Ethernet frame and
have it reach the destination. The destination isn't on the same
Ethernet network! So your computer instead forwards the packet to its
_default gateway_, that is, the router of last resort. If your computer
doesn't have a routing table entry for the destination subnet in
question, it sends it to the default gateway, which forwards it to the
greater Internet.

Let's look at an example routing table from my Linux machine, which in
this example has been assigned address `192.168.1.230`.

<!-- CAPTION: Example Routing Table -->
||Source|Destination|Gateway|Device|
|-|-|-|-|-|
|1|`127.0.0.1`|`127.0.0.1`||`lo`|
|2|`127.0.0.1`|`127.0.0.0/8`||`lo`|
|3|`127.0.0.1`|`127.255.255.255`||`lo`|
|4|`192.168.1.230`|`192.168.1.230`||`wlan0`|
|5|`192.168.1.230`|`192.168.1.0/24`||`wlan0`|
|6|`192.168.1.230`|`192.168.1.255`||`wlan0`|
|7|`192.168.1.230`|`default`|`192.168.1.1`|`wlan0`|

Let's take a look at connecting from `localhost` to `localhost`
(`127.0.0.1`). In that case, the OS looks up what route matches and
sends the data on the corresponding interface. In this case, that's the
_loopback_ interface (`lo`), a "fake" interface that the OS pretends is
a network interface. (For performance reasons.)

But what if we send data from `127.0.0.1` to anything on the
`127.0.0.0/8` subnet? It uses the `lo` interface as well. And the same
thing happens if we send data to the `127.255.255.255` broadcast address
(on the `127.0.0.0/8` subnet).

The other entries are more interesting. Remember as you look at these
that my machine has been assigned `192.168.1.230`.

So if we look at line 4, above, we're looking at the case where I send
from my machine to itself. This is like `localhost` except I'm
deliberately using the IP attached to my wireless LAN device,
`wlan0`. So the OS is going to use that system, but is smart enough to
not bother sending it over the wire--after all, _this_ is the
destination.

After that, we have the case where we're sending to any other host on
the `192.168.1.0/24` subnet. So this is like my sending from my machine
as `192.168.1.230` to another machine, for example `192.168.1.22`, say.
Or any other machine on that subnet.

And then on line 6 we have a broadcast address for the LAN, which also
goes out on the WiFi device.

But what if all that fails? What if I'm sending from `192.168.1.230` to
`203.0.113.37`? That's not an IP or subnet listed on my destinations in
my routing table.

This is what line 7 is for: the _default gateway_. This is where a
router sends packets if it doesn't know how to otherwise route them.

At your house, this is the router that you got from your ISP. Or that
you bought and installed yourself if you were so-inclined.

So when I ping `example.com` (`93.184.216.34` as of this writing) from
my home computer, those packets get sent to my default gateway because
that IP and its corresponding subnet don't appear in my computer's
routing table.

(And they don't appear in my default gateway's routing table, either. So
it forwards them to its default route.)

## Routing Algorithm

Let's assume now that the routing protocols have done their job and all
the routers have the information they need to know where to forward
packets along the way.

When the packet arrives, the router follows a sequence of steps to
decide what to do with it next. (For this discussion we'll ignore the
loopback device and assume all traffic is on an actual network.)

![IP Routing Algorithm]()

* If the destination IP is on the local network attached to the router
  * Deliver the packet on the link (physical layer, e.g. Ethernet)
* Else If the routing table has an entry for the destination IP's network
  * Deliver to the next router toward that destination
* Else If a default route exists:
  * Deliver to default gateway router
* Else
  * Drop packet
  * Send an
    [ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol)
    "Destination Unreachable" error message back to the sender

If multiple matching routes exist, the one with the longest subnet mask
is chosen.

If there is still a tie, the one with the loewst metric is used.

If there is still a tie, ECMP (Equal Cost Multi-Path) routing can be
used--send the packet down both paths.

## Routing Example

Let's run some examples. [Download a PDF of this diagram for greater
clarity]().

In this diagram there are lot of missing things. Notably that every
router interface has an IP address attached to it.

Also note that, for example, Router 1 is directly connected to 3
subnets, and it has an IP address on each of them.

![Network Diagram]()

Trace the route of these packets:

<!-- Example Sources and Destinations -->
|Source|Destination|
|-|-|-|
|`10.1.23.12`|`10.2.1.16`||
|`10.1.99.2`|`10.1.99.6`||
|`192.168.2.30`|`8.8.8.8`||
|`10.2.12.37`|`192.168.2.12`|`10.0.0.0/8` and `192.168.0.0/16` are private networks and don't get routed over the outside Internet|
|`10.1.17.22`|`10.1.17.23`||
|`10.2.12.2`|`10.1.23.12`||

## Routing Loops and Time-To-Live

It should be apparent that it would be possible to set up a loop where a
packet traveled in a circle forever.

To solve this problem, IP has a field in its header: _time to live_
(TTL). This is a one-byte counter that starts at 255 and gets
decremented every time a router forwards a packet.

When the counter reaches zero, the packet is discarded and an
[ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol)
"Time Exceeded" error message is returned to the sender.

So the most hops that a packet can make is 255, even in a loop.

The `traceroute` utility function works by sending a packet toward a
destination with a TTL of 1 and seeing who sends back the ICMP message.
Then it sends out another with a TTL of 2 and sees who responds. And it
keeps increasing the TTL on subsequent packets until the ultimate
destination responds.

## The Broadcast Address

There's a special address in IPv4 called the _broadcast address_. This
is an address that sends a packet to every computer on the LAN.

This is an address on a subnet with all the host bits set to `1`.

For example:

<!-- CAPTION Broadcast Addresses -->
|Subnet|Subnet Mask|Broadcast Address|
|-|-|-|
|`10.20.30.0/24`|`255.255.255.0`|`10.20.30.255`|
|`10.20.0.0/16`|`255.255.0.0`|`10.20.255.255`|
|`10.0.0.0/8`|`255.0.0.0`|`10.255.255.255`|

There's also _the_ broadcast address: `255.255.255.255`. This goes to
everyone...

...And by everyone I mean everyone on the LAN. None of these broadcast
packets make it past the router, not even `255.255.255.255`. The routers
don't forward them anywhere.

The world would be a very noisy place if they did.

One of the main uses for it is when you first open your laptop on WiFi.
The laptop doesn't know it's IP, the subnet mask, the default gateway,
or even whom to ask. So it sends out a broadcast packet to
`255.255.255.255` with a
[DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)
packet asking for that information. A DHCP server is listening and can
then reply with the info.

And since you're all wondering: IPv6 doesn't have a broadcast address;
its obviated by a thing called IPv6 multicast. Same idea, just more
focused.

## Reflect

* What is the difference between an interior gateway protocol and an
  external gateway protocol?

* What is the goal of a routing protocol in general?

* What's an example of a place where an interior gateway protocol would
  be used? And exterior?

* What does a router use its routing table to determine?

* What does an IP router do next with a packet if the destination IP
  address is not on one of its local subnets?

* Why would a process send anything to the broadcast address?
