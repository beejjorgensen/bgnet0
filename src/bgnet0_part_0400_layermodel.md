# The Layered Network Model

Before we get started, here are some terms to know:

* **IP Address** -- historically 4-byte number uniquely identifying your
  computer on the Internet. Written in dots-and-numbers notation, like
  so: `198.51.100.99`.

  These are IP version 4 ("IPv4") addresses. Typically "v4" is implied
  in the absense of any other version identifier.

* **Port** -- Programs talk through ports, which are numbered 0-65535
  and are associated with the TCP or UDP protocols.

  Since multiple programs can be running on the same IP address, the
  port provides a way to uniquely identify those programs on the
  network.

  For example, it's very common for a webserver to listen for incoming
  connections on port 80.

  Publishing the port number is really important for server programs
  since client programs need to know where to connect to them.

  Clients usually let the OS choose an unused port for them to use since
  no one tries to connect to clients.

  In a URL, the port number is after a colon. Here we try to connect to
  `example.com` on port `3490`: `http://example.com:3490/foo.html` 

  Ports under 1024 need root/administrator privileges to bind to (but
  not to connect to).

* **TCP** -- Transmission Control Protocol, responsible for reliable,
  in-order data transmission. From a higher-up perspective, makes a
  packet-switched network feel more like a circuit-switched network.

  TCP uses port numbers to identify senders and receivers of data.

  This protocol was invented in 1974 and is still in extremely heavy use
  today.

* **UDP** -- sibling of TCP, except lighter weight. Doesn't guarantee
  data will arrive, or that it will be in order, or that it won't be
  duplicated. If it arrives, it will be error-free, but that's all you
  get.

* **IPv6 Address** -- Four bytes isn't enough to hold a unique address,
  so IP version 6 expands the address size considerably to 16 bytes.
  IPv6 addresses look like this: `::1` or `2001:db8::8a2e:370:7334`, or
  even bigger.

* **NAT** -- Network Address Translation. A way to allow organizations
  to have private subnets with non-globally-unique addresses that get
  translated to globally-unique addresses as they pass through the
  router.

  Private subnets commonly start with addresses `192.168.x.x` or
  `10.x.x.x`.

* **Router** -- A specialized computer that forwards packets through the
  packet switching network. It inspects destination IP addresses to
  determine which route will get the packet closer to its goal.

* **IP** -- Internet Protocol. This is responsible for identifying
  computers by IP address and using those addresses to route data to
  recipients through a variety of routers.

* **LAN** -- Local Area Network. A network where all the computers are
  effectively directly connected, not via a router.

* **Interface** -- physical networking hardware on a computer. A
  computer might have a number of interfaces. Your computer likely has
  two: a wired Ethernet interface and a wireless Ethernet interface. 

  A router might have a large number of interfaces to be able to route
  packets to a large number of destinations. Your home router probably
  only has two interfaces: one facing inward to your LAN and the other
  facing outward to the rest of the Internet.

  Each interface typically has one IP address and one MAC address.

  The OS names the interfaces on your local machine. They might be
  something like `wlan0` or `eth2` or something else. It depends on the
  hardware and the OS.

* **Header** -- Some data that is prepended to some other data by a
  particular protocol. The header contains information appropriate for
  that protocol. A TCP header would include some error detection and
  correction information and a source and destination port number. IP
  would include the source and destination IP addresses. Ethernet would
  include the source and destination MAC addresses. And an HTTP response
  would include things like the length of the data, the date modified,
  and whether or not the request was successful.

  Putting a header in front of the data is analogous to putting your
  letter in an envelope in the snail-mail analogy. Or putting that
  envelope in another envelope.

  As data moves through the network, additional headers are added and
  removed. Typically only the top-most (front-most?) header is removed
  or added in normal operation, like a stack. (But some software and
  hardware peeks deeper.)

  **Network Adapter** -- Another name for "network card", the hardware
  on your computer that does network stuff.

  **MAC Address** -- Ethernet interfaces have MAC addresses, which take
  the form `aa:bb:cc:dd:ee:ff`, where the fields are random-ish one-byte
  hex numbers. MAC addresses are 6 bytes long, and must be unique on the
  LAN. When a network adapter is manufactured, it is given a unique MAC
  address that it keeps for life, typically.

## The Layered Network Model

When you send data over the Internet, that data is _encapsulated_ in
different layers of protocols.

The layers of the conceptual layered network model correspond to various
classes of protocols.

And those protocols are responsible for different things, e.g.
describing data, preserving data integrity, routing, local delivery,
etc.

So it's a little chicken-and-eggy, because we can't really discuss one
without the other.

Best just to dive in and take a look at protocols layering headers on
top of some data.

## An Example of Layering of Protocols on Data

Let's consider what happens with an HTTP request.

1. The web browser builds the HTTP request that looks like this:

   ``` {.default}
   GET / HTTP/1.1
   Host: example.com
   Connection: close

   ```

   And that's all the browser cares about. It doesn't care about IP
   routing or TCP data integrity or Ethernet.

   It just says "Send this data to that computer on port 80".

2. The OS takes over and says, "OK, you asked me to send this over a
   stream-oriented socket, and I'm going to use the TCP protocol to do
   that and insure all the data arrives intact and in order."

   So the OS takes the HTTP data and wraps it in a TCP header which
   includes the port number.

3. And then the OS says, "And you wanted to send it to this remote
   computer whose IP address is 198.51.100.2, so we'll use the IP
   protocol to do that."

   And it takes the entire TCP-HTTP data and wraps it up an an IP
   header. So now we have data that looks like this: IP-TCP-HTTP.

4. After that, the OS takes a look at its routing table and decides
   where to send the data next. Maybe the web server is on the LAN,
   conveniently. More likely, it's somewhere else, so the data would be
   sent to the router for your house destined for the greater Internet.

   In either case, it's going to send the data to a server on the LAN,
   or to your outbound router, also on the LAN. So it's going to a
   computer on the LAN.

   And computers on the LAN have an Ethernet address (AKA _MAC
   address_--which stands for "Media Access Control"), so the sending OS
   looks up the MAC address that corresponds to the next destination IP
   address, whether that's a local webserver or the outbound router.
   (This happens via a lookup in something called the _ARP Cache_, but
   we'll get to that part of the story another time.)

   And it wraps the whole IP-TCP-HTTP packet in an Ethernet header, so
   it becomes Ethernet-IP-TCP-HTTP. The web request is still in there,
   buried under layers of protocols!

5. And finally, the data goes out on the wire (even if it's WiFi, we
   still say "on the wire").

The computer with the destination MAC address, listening carefully, sees
the Ethernet packet on the wire and reads it in. (Ethernet packets are
called _Ethernet frames_.)

It strips off the Ethernet header, exposing the IP header below it. It
looks at the destination IP address.

1. If the inspecting computer is a server and it has that IP address,
   its OS strips off the IP header and looks deeper. (If it doesn't have
   that IP address, something's wrong and it discards the packet.)

2. It looks at the TCP header and does all the TCP magic needed to make
   sure the data isn't corrupted. If it is, it replies back with the
   magic TCP incantations, saying, "Hey, I need you to send that data
   again, please."

   Note that the web browser or server never knows about this TCP
   conversation that's happening. It's all behind the scenes. For all it
   can see, the data is just magically arriving intact and in order.

   The reason is that they're on a higher layer of the network. They
   don't have to worry about routing or anything. The lower layers take
   care of it.

3. If everything's good with TCP, that header gets stripped and the OS
   is left with the HTTP data. It wakes up the process (the web server)
   that was waiting to read it, and gives it the HTTP data.

But what if the destination Ethernet address was an intermediate router?

1. The router strips off the Ethernet frame as always.

2. The router looks at the destination IP address. It consults its
   routing table and decides to which interface to forward the packet.

3. It sends it out to that interface, which wraps it up in another
   Ethernet frame and sends it to the next router in line.

   (Or maybe it's not Ethernet! Ethernet is a protocol, and there are
   other low-level protocols in use with fiber optic lines and so on.
   This is part of the beauty of these layers of abstraction--you can
   switch protocols partway through transmission and the HTTP data above
   it is completely unaware that any such thing has happened.)

## The Internet Layer Model

Let's start with the easier model that splits this transmission up into
different layers from the top down. (Note that the list of protocols is
far from exhaustive.)

<!-- CAPTION: Internet Layered Network Model -->
|Layer|Responsibility|Example Protocols|
|:-:|-|-|
|Application|Structured application data|HTTP, FTP, TFTP, Telnet, SSH, SMTP, POP, IMAP|
|Transport|Data Integrity, packet splitting and reassembly|TCP, UDP|
|Internet|Routing|IP, IPv6, ICMP|
|Link|Physical, signals on wires|Ethernet, PPP, token ring|

You can see how different protocols take on the responsibilities of each
layer in the model.

Another way to think of this is that all the programs that implement
HTTP or FTP or SMTP can use TCP or UDP to transmit data. (Typically all
sockets programs and applications you write that implement any protocol
will live at the application layer.)

And all data that's transmitted with TCP or UDP can use IP or IPv6 for
routing.

And all data that uses IP or IPv6 for routing can use Ethernet or PPP,
etc. for going over the wire.

And as a packet moves down through the layers before being transmitted
over the wire, the protocols add their own headers on top of everything
else so far.

This model is complex enough for working on the Internet. You know what
they say: as simple as possible, but no simpler.

But there might be other networks in the Universe that aren't the
Internet, so there's a more general model out there that folks sometimes
use: the OSI model.

## The ISO OSI Network Layer Model

This is imporant to know if you're taking a certification test or if
you're going into the field as more than a regular programmer.

The Internet Layer Model is a special case of this more-detailed model
called the ISO OSI model. (Bonus points for being a palindrome.) It's
the International Organization for Standardization Open Systems
Interconnect model. I know that "ISO" is not a direct English
abbreviation for "International Organization for Standardization", but I
don't have enough global political influence to change that.

Coming back to reality, the OSI model is like the Internet model, but
more granular.

The Internet model maps to the OSI model, like so, with a single layer
of the Internet model mapping to multiple layers of the OSI model:

<!-- CAPTION: OSI to Internet Layer Mapping -->
|ISO OSI Layer|Internet Layer|
|:-:|:-:|
|Application|Application|
|Presentation|Application|
|Session|Application|
|Transport|Transport|
|Network|Network|
|Data link|Link|
|Physical|Link|

And if we look at the OSI model, we can see some of the protocols that
exist at those various layers, similar to what we saw with the Internet
model, above.

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

We're going to stick with the Internet model for this course since it's
good enough for 99.9% of the network programming work you'd ever be likely
to do. But please be aware of the OSI model if you're going into an
interview for a network-specific programming position.

## Reflect

* When a router sees an IP address, how does it know where to forward
  it?

* If an IPv4 address is 4 bytes, roughly how many different computers
  can that represent in total, assuming each computer has a unique IP
  address?

* Same question, except for IPv6 and its 16-byte addresses?

* Bonus question for stats nerds: The odds of winning the super lotto
  jackpot are approximately 300 million to 1. What are the odds of
  randomly picking my pre-selected 16-byte (128-bit) number?

* Speculate on why IP is above TCP in the layered model. Why does the
  TCP header go on before the IP header and not the other way around?

* If UDP is unreliable and TCP is reliable, speculate on why one might
  ever use UDP.


