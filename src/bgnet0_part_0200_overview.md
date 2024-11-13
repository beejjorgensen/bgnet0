# Networking Overview

The Big Idea of networking is that we're going to get arbitrary bytes of
data exchanged between two (or more) computers on the Internet or LAN.

So we need:

* A way of identifying the source and destination computers.
* A way of maintaining data integrity.
* A way of routing data from one computer to another (out of billions).

And we need all the hardware and software support to make this happen.

Let's take a look at two basic kinds of communications networks.

## Circuit Switched

Don't be alarmed! The Internet doesn't use this type of network (at
least not as far as it is aware). So you can read the rest of this
section with the confident knowledge that you won't have to use it
again.

For this type of network, visualize an old-school telephone operator
(like a human operator). Back before you could dial numbers directly, a
call went something like this:

You'd pick up the receiver and turn a crank on it to generate an
electrical signal that rang a bell at the other end of the phone line,
which was in some telephone exchange office somewhere.

An operator would hear the bell and pick up the other end and say
something appropriate like, "Operator."

And you'd tell (like with your voice) the operator what number you
wanted to connect to.

Then the operator would physically plug jumper wires into the
switchboard in front of them to physically complete an electrical
connection between your phone and the phone of the person you wanted to
call. You'd have a direct wire from your phone to their phone.

This scales very poorly. And if you had to make a long distance call,
that cost extra, because an operator would have to call another operator
over a limited number of long-distance lines.

Eventually, people figured out they could replace the human operators
with electro-mechanical relays and switches that you could control by
sending carefully coded signals down the line, either electrical pulses
(sent by old rotary dial phones) or the still-recognizable "touch"
tones.

But we still had problems:

* A dedicated circuit was needed for every call.
* Even if you sat there quietly saying nothing, the circuit was still
  occupied and couldn't be used by anyone else.
* Multiple people couldn't use the same line at the same time.
* There were a limited number of wires you could string up.

So the Internet took a different approach.

## Packet Switched

In a _packet switched_ network, the data you want to send is split up
into individual packets of varying numbers of bytes. If you want to send
83,234 bytes of data, that might be split up into 50 or 60 packets.

Then each of these packets are individually sent over the lines as space
permits.

Imagine little packets of data from computers all over North America
arriving at a router at the edge of the Atlantic Ocean which sends them,
one at a time, over a thousands-of-miles-long undersea cable to Europe.

Once the packets arrive at their destination computer, that computer
reconstructs the data from the individual packets.

This is analogous to writing a snail-mail letter and putting it in the
post. It ends up in a truck with a bunch of other pieces of mail that
aren't going to the same place as yours.

The post office routes the letters through the appropriate mailing
facilities until they arrive.

Maybe your letter gets on a plane headed for the opposite side of the
country with a bunch of other letters. And when the plane arrives, those
letters might part, with some going north and some going south.

They don't use a whole plane for a single letter--the letters are like
packets, and they get switched from one transportation medium to
another.

In the same way, packets of data on the Internet will move from computer
to computer, sharing the lines between those computers with other
traffic, until they finally get where they're going.

And this affords some great advantages in a computer network:

* You don't need a dedicated circuit between every communicating pair of
  computers (this would likely be a physical impossibility if we wanted
  to support the amount of traffic we currently have today).

* Multiple computers can all use the same wire to send data at the
  "same" time. (The packets actually go one at a time, but they're
  interleaved so it looks simultaneous.)

* A wire can be utilized to full capacity; there's no "dead air" that
  goes unused if someone wants to use it. One computer's silence is
  another computer's opportunity to transmit on the same wire.

## Client/Server Architecture

You know this from using the web--you've heard of web servers.

A _server_ is a program that _listens_ for incoming connections,
_accepts_ them, and then typically receives a request from the _client_
and sends back a response to the client.

Actual conversations between the server and client might be far more
complex depending on what the server does.

But both the client and server are network programs. The practical
difference is the server is the program sitting there waiting for
clients to call.

Typically one server exists for many clients. Many servers might exist
in a _server farm_ to serve many, many, many clients. Think about how
many people use Google at the same time.

## The OS, Network Programming, and Sockets

The network is hardware, and the OS controls all access to hardware. So
if you want to write software to use the network, you have to do it
through the OS.

Historically, and modernly, this was done using an API called the
_sockets_ API that was pioneered on Unix.

The Unix sockets API is very general purpose, but one of the many things
it can do is give you a way to read and write data over the Internet.

Other languages and operating systems have added the same Internet
functionality over time, and many of them use different calls in their
APIs. But as an homage to the original, many of these APIs are still
called "sockets" APIs even if they don't match the original.

If you want to use the original sockets API, you can do it programming
with C in Unix.

## Protocols

You know that conversation that the client and server have? It's written
down very specifically what bytes get sent when and from and to whom.
You can't just send any old data to a web server--it has to be wrapped
up a certain way.

Just like you can't take a letter, wrap it up in aluminum foil with no
address, and expect the post office to deliver it to your intended
recipient. That's breaking post office _protocol_.

Both the sender and recipient have to be speaking the same protocol for
correct communication to occur.

> "Thank you for calling The Pizza Restaurant. Can I help you?"
> "Would you like fries with that?"
>
> A person calling a pizza restaurant breaks protocol.

There are many protocols, and we'll cover a few of them in detail later.
These were invented by people to solve different sorts of problems. If
you need to pass data between two specialized programs you write, you'll
have to define a protocol for that, too!

Here are some common ones you might have heard of:

* TCP - used to transmit data reliably.
* UDP - used to transmit data quickly and unreliably.
* IP - used to route packets over the network from one computer to
  another.
* HTTP - used to get web pages and make other web requests.
* Ethernet - used to send data over a LAN.

As we'll see in a moment, these protocols "live" at different layers of
the network software.

## Network Layers and Abstraction

Here's a quick overview of what happens when data goes out on the
network. We'll cover this in much more detail in the coming modules.

1. A user program says, "I want to send the bytes 'GET / HTTP/1.1' to
   that web server over there."  (Servers are identified by _IP address_
   and a _port_ on the Internet--more on that later.)

2. The OS takes the data and wraps it up in a _header_ (that is,
   prepends some data) that provides error detection (and maybe
   ordering) information. The exact structure of this header would be
   defined by a protocol such as TCP or UDP.

3. The OS takes all of _that_, and wraps it up in another header that
   helps with routing. This header would be defined by the IP protocol.

4. The OS hands all that data to the network interface card (the
   _NIC_--the piece of hardware that's responsible for networking).

5. The NIC wraps all _that_ data up into another header that's defined
   by a protocol such as Ethernet that helps with delivery on the LAN.

6. The NIC sends the entire, multiply-wrapped data out over the wire,
   or over the air (with WiFi).

When the receiving computer gets the packet, the reverse process
happens. Its NIC strips the Ethernet header, the OS makes sure the IP
address is correct, figures out which program is listening on that port,
and sends it the fully unwrapped data.

All these different layers that do all this wrapping are together called
the _protocol stack_. (This is a different usage of the word "stack"
than the stack abstract data type.)

This works well because each layer is responsible for different parts of
the process, e.g. one layer handles data integrity, and another handles
routing the packet over the network, and another handles the data itself
that is being transmitted between the programs. And each layer doesn't
care about what the layers below it are doing with the data.

It's that last concept that's really important: when data is going over
WiFi, the WiFi hardware doesn't even care what the data is, if it's
Internet data or not, how integrity is assured (or not). All WiFi cares
about is getting a big chunk of data transmitted over the air to another
computer. When it arrives at the other computer, that computer will
strip off the Ethernet stuff and look deeper in the packet, deciding
what to do with it.

And since the layers don't care what data is encapsulated below them,
you can swap out protocols at various layers and still have the rest of
them work. So if you're writing a program at the top layer (where we
tend to write them most commonly), you don't care what's happening at
the layers below that. It's Somebody Else's Problem.

For example, you might be getting a web page with HTTP/TCP/IP/Ethernet,
or you might be transmitting a file to another computer with
TFTP/UDP/IP/Ethernet. IP and Ethernet work fine in both cases, because
they are indifferent about the data they are sending.

There are many, many details omitted from this description, but we're
still in high-level overview land.

## Wired versus Wireless

When we're talking about LANs, we can think about network programming as
if these two things were the same:

* Computers on a LAN connected by physical Ethernet cables[1].
* Computers on a LAN all connected to the same WiFi access point.

Turns out they both use the Ethernet protocol for low-level
communication.

So when we say the computers are on the same LAN, we mean they are
either wired together or they are using the same WiFi access point.

[1] It's a bit wrong to call them "Ethernet cables" because they are
just wires, and Ethernet is a protocol that effectively defines patterns
of electricity that go over those wires. But what I mean is, "a cable
that is commonly used with Ethernet".

## Reflect

* Is the Internet circuit-switched or packet-switched?

* What is the relationship between a client program and a server
  program?

* What role does the OS play when you're writing networked programs?

* What is a protocol?

* What are the reasons for having a protocol stack and data
  encapsulation?

* What are the practical differences between a WiFi network and a wired
  network?
