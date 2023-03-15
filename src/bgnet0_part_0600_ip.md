# The Internet Protocol (IP)

This protocol is responsible for routing packets of data around the
Internet, analogous to how the post office is responsible for routing
letters around the mail network.

Like with the post office, data on the Internet has to be labeled with a
source and destination address, called the _IP address_.

The IP address is a sequence of bytes that uniquely identifies every
computer on the Internet.

## Terminology

* **Host** - another name for "computer".

## Two Common Versions

There are two commonly used versions of IP: version 4 and version 6.

IP version 4 is referred to as "IPv4" or just plain "IP".

IP version 6 is usually explicitly specified as "IPv6".

You can tell the difference between the two by glancing at an IP
address:

* Version 4 IP address example: 192.168.1.3

* Version 6 IP address example: fe80::c2b6:f9ff:fe7e:4b4

The main difference is the number of bytes that make up the address
space. IPv4 uses 4 bytes per address, and IPv6 uses 16 bytes.

## Subnets

Every IP address is split into two portions.

The initial bits of the IP address identify individual networks.

The trailing bits of an IP address identify individual hosts (i.e.
computers) on that network.

These individual networks are called _subnets_ and the number of hosts
they can support depends on how many bits they're reserved for
identifying hosts on that subnet.

As a contrived non-Internet example, let's look at an 8-bit "address",
and we'll say the first 6 bits are the network number and the last 2
bits are the host number.

So an address like this:

``` {.default}
00010111
```

is split into two parts (because we said the first 6 bits were the
network number):

``` {.default}
Network  Host
-------  ----
000101   11
```

So this is network 5 (`101` binary), host 3 (`11` binary).

The network part always comes before the host part.

Note that if there are only two "host" bits, there can only be 4 hosts
on the network, numbered 0, 1, 2, and 3 (or `00`, `01`, `10`, and `11`
in binary).

And with IP, it would actually only be two hosts, because hosts with
all zero bits or all one bits are reserved.

The next chapters will look at specific subnet examples for IPv4 and
IPv6. The important part now is that each address is split into network
and host parts, with the network part first.

## Additional IP-layer Protocols

There are some related protocols that also work in concert with IP and
at the same layer in the network stack.

* **ICMP**: Intenet Control Message Protocol, a mechanism for
  communicating IP nodes to talk about IP control metadata with one
  another.

* **IPSec**: Internet Protocol Security, encryption and authentication
  functionality. Commonly used with VPNs (Virtual Private Networks).

Users commonly interface with ICMP when using the `ping` utility. This
uses ICMP "echo request" and "echo response" messages.

The `traceroute` utility uses ICMP "time exceeded" messages to find out
how packets are being routed.

## Private Networks

There are private networks hidden behind routers that do not have
globally unique IP addresses on their machines. (Though they do have
unique addresses within the LAN itself.)

This is made possible through the magic of a mechanism called NAT
(Network Address Translation). But this is a story for the future.

For now, let's just pretend all our addresses are globally unique.

## Static versus Dynamic IP Addresses, and DHCP

If you have clients hitting your website, or you have a server that you
want to SSH into repeatedly, you'll need a _static IP_. This means you
get a globally-unique IP address assigned to you and it never changes.

This is like having a house number that never changes. If you need
people to be able to find your house repeatedly, this needs to be the
case.

But since there are a limited number of IPv4 addresses, static IPs cost
more money. Often an ISP will have a block of IPs on a subnet that they
_dynamically_ allocate on-demand.

This means when you reboot your broadband modem, it might end up with a
different public-facing IP address when it comes back to life. (Unless
you've paid for a static IP.)

Indeed, when you connect your laptop to WiFi, you also typically get a
dynamic IP address. Your computer connects to the LAN and broadcasts a
packet saying, "Hey, I'm here! Can anyone tell me my IP address? Pretty
please with sugar on top?"

And this is OK because people aren't generally trying to connect to
servers on your laptop. It's usually the laptop that's connecting to
other servers.

How does it work? On one of the servers on the LAN is a program that is
listening for such requests, which conform to DHCP (the _Dynamic Host
Configuration Protocol_). The DHCP server keeps track of which IP
addresses on the subnet are already allocated for use, and which are
free. It allocates a free one and sends back a DHCP response that has
your laptop's new IP address, as well as other data about the LAN your
computer needs (like subnet mask, etc.).

If you have WiFi at home, you very likely already have a DHCP server.
Most routers come from your ISP with DHCP already set up, which is how
your laptop gets its IP address on your LAN.

## Reflect

* How many times more IPv6 addresses are there than IPv4 addresses?

* Applications commonly also implement their own encryption (e.g. ssh or
  web browsers with HTTPS). Speculate on the advantages or disadvantages
  for having IPSec at the Internet layer instead of doing encryption at
  the Application layer.

* If subnet reserved 5 bits to identify hosts, how many hosts can it
  support? Don't forget that all-zero-bits and all-one-bits for the host
  are reserved.

* What is the benefit to having a static IP? How does it relate to DNS?
