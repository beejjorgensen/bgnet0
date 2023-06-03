# The Internet Protocol version 4

This is the first popular version of the Internet Protocol, and it lives
to this day in common use.

## IP Addresses

An IPv4 address is written in "dots and numbers" notation, like so:

``` {.default}
198.51.100.125
```

It's always four numbers. Each number represents a byte, so it can go
from 0 to 255 (`00000000` to `11111111` binary).

This means that every IPv4 address is four bytes (32 bits) in size.

## Subnets

The entire space of IP addresses is split up into _subnets_. The first
part of the IP address indicates the subnet number we're talking about.
The remaining part indicates the computer on that subnet in question.

And how many bits "the first part of the IP" constitutes is variable.

When you set up a network with public-facing IP addresses, you are
allocated a subnet by whomever you are paying to provide you with a
connection. The more hosts your subnet supports, the more expensive it
is.

So you might say, "I need 180 IP static IP addresses."

And your provider says, OK, that means you'll have 180 IPs and 2 reserved
(0 and the highest number), so 182 total. We need 8 bits to represent the
numbers 0-255, which is the smallest number of bits that includes 182.

And so they allocate you a subnet that has 24 network bits and 8 host
bits.

They could write out something like:

``` {.default}
Your subnet is 198.51.100.0 and there are 24 network bits and 8 host
bits.
```

But that's really verbose. So we use slash notation:

``` {.default}
198.51.100.0/24
```

This tells us that 24 bits of the IP address represent the network
number. (And therefore `32-24=8` bits represent the host.) But what does
that mean?

Drawing it out:

``` {.default}
24 network bits
----------
198.51.100.0
           -
         8 host bits
```

Or converting all those numbers to binary:

``` {.default}
       24 network bits         | 8 host bits
-------------------------------+---------
11000110 . 00110011 . 01100100 . 00000000
     198         51        100          0
```

The upshot is that every single IP on our make-believe network here is
going to start with `198.51.100.x`. And that last byte is going to
indicate which host we're talking about.

Here are some example IPs on our network:

``` {.default}
198.51.100.2
198.51.100.3
198.51.100.4
198.51.100.30
198.51.100.212
```

But these two addresses have special meaning (see below):

``` {.default}
198.51.100.0     Reserved
198.51.100.255   Broadcast (see below)
```

but other than those, we can use the other IPs as we see fit.

Now, I deliberately chose an example there where the subnet ended on a
byte boundary because it's easier to see if the entire last byte is the
host number.

But there's no law about that. We could easily have a subnet like this:

``` {.default}
198.51.100.96/28
```

In that case we have:

``` {.default}
           28 network bits           | 4 host bits
-------------------------------------+----
11000110 . 00110011 . 01100100 . 0110 0000
     198         51        100          96
```

and we could only fill those last 4 bits with different numbers to
represent our hosts.

`0000` and `1111` are reserved and broadcast, leaving us with 14 more we
could use for host numbers.

For example, we could fill in those last [4
bits](https://en.wikipedia.org/wiki/Nibble) with host number 2 (which is
`0010` binary):

``` {.default}
           28 network bits           | 4 host bits
-------------------------------------+----
11000110 . 00110011 . 01100100 . 0110 0010
     198         51        100          98
```

Giving the IP address `198.51.100.98`.

All the IP addresses on this subnet are, exhaustively `198.51.100.96`
through `198.51.100.111` (though these first are last IPs are reserved
and broadcast, respectively).

Finally, if you have a subnet you own, there's nothing stopping you for
further subnetting it down--declaring that more bits are reserved for
the network portion of the address.

ISPs (Internet Service Providers, like your cable or DSL company) do
this all the time. They've given a big subnet with, say, 12 network bits
(20 host bits, for 1 million possible hosts). And they have customers
who want their own subnets. So the ISP decides the next 9 bits (for
example) are going to be used to uniquely identify additional subnets
within the ISP's subnet. And it sells those to customers, and each
customer gets 11 bits for hosts (supporting 2048 hosts).

``` {.default}
  ISP network  |   Subnets  |    Hosts
   (12 bits)   |  (9 bits)  |  (11 bits)
---------------+------------+--------------
11010110 . 1100 0101 . 11011 001 . 00101101   [example IP]
```

But it doesn't even stop there, necessarily. Maybe one of those
customers you sold an 11-bit subnet to wants to further subdivide
it--they can add more network bits to define their own subnets. Of
course, every time you add more network bits, you're taking away from
the number of hosts you can have, but that's the tradeoff you have to
make with subnetting.

## Subnet Masks

Another way of writing subnet is with a _subnet mask_. This is a number
that when bitwise-ANDed with any IP address will give you the subnet
number.

What does that mean? And why?

The subnet mask is also written with dots-and-numbers notation, and
looks like an IP address with all the subnet bits set to `1`.

For example, if we have the subnet `198.51.100.0/24`, that means we
have:

``` {.default}
       24 network bits         | 8 host bits
-------------------------------+---------
11000110 . 00110011 . 01100100 . 00000000
     198         51        100          0
```

Putting a `1` in for all the network bits, we end up with:

``` {.default}
       24 network bits         | 8 host bits
-------------------------------+---------
11111111 . 11111111 . 11111111 . 00000000
     255        255        255          0
```

So the subnet mask for `198.51.100.0/24` is `255.255.255.0`. It's the
same subnet mask for _any_ `/24` subnet.

The subnet mask for a `/16` subnet has the first 16 bits set to `1`:
`255.255.0.0`.

But why? Turns out a router can take any IP address and quickly
determine its destination subnet by ANDing the IP address with the
subnet mask.


``` {.default}
         24 network bits         | 8 host bits
  -------------------------------+---------
  11000110 . 00110011 . 01100100 . 01000011    198. 51.100.67
& 11111111 . 11111111 . 11111111 . 00000000  & 255.255.255. 0
-------------------------------------------  ----------------
  11000110 . 00110011 . 01100100 . 00000000    198. 51.100. 0
```

And so the subnet for the IP address `198.51.100.67` with subnet mask
`255.255.255.0` is `198.51.100.0`.

## Historic Subnets

_(This information is only included for historical interest.)_

Before the idea that any number of bits could be reserved for the
network, subnets were split into 3 main classes:

* **Class A** - Subnet mask `255.0.0.0` (or `/8`), supports 16,777,214 hosts
* **Class B** - Subnet mask `255.255.0.0` (or `/16`), supports 65,534 hosts
* **Class C** - Subnet mask `255.255.255.0` (or `/24`) supports 254 hosts

The problem was that this caused a really uneven distribution of
subnets, which some large companies getting 16 million hosts (that they
didn't need), and there was no subnet class that supported a sensible
number of computers, like 1,000.

So we switched to the more-flexible "any number of bits in the mask"
approach.

## Special Addresses

There are a few common addresses that are worth noting:

* **127.0.0.1** - this is the computer you are on now. It's often
  mapped to the name `localhost`.

* **0.0.0.0** - Reserved. Host 0 on any subnet is reserved.

* **255.255.255.255** - Broadcast. Intended for all hosts on a subnet.
  Though it seems like this would broadcast to the entire Internet,
  routers don't forward packets intended for this address.

  You can also broadcast to your local subnet by sending to the host
  with all bits set to 1. For example, the subnet broadcast address for
  `198.51.100.0/24` is `198.51.100.255`.

## Special Subnets

There are some reserved subnets you might come across:

* **10.0.0.0/8** - Private network addresses (_very common_)
* **127.0.0.0/8** - This computer, via the _loopback_ device.
* **172.16.0.0/12** - Private network addresses
* **192.0.0.0/24** - Private network addresses
* **192.0.2.0/24** - Documentation
* **192.168.0.0/16** - Private network addresses (_very common_)
* **198.18.0.0/15** - Private network addresses
* **198.51.100.0/24** - Documentation
* **203.0.113.0/24** - Documentation
* **233.252.0.0/24** - Documentation

You'll find your home IPs are in one of the "Private" address ranges.
Probably `192.168.x.x`.

Any documentation that you write that requires example (not real) IP
addresses should use any of the ones marked "Documentation", above.

## Reflect

* `192.168.262.12` is not a valid IP address. Why?

* Reflect on some of the advantages of the subnet concept as a way of
  dividing the global address space.

* What is your computer's IPv4 address and subnet mask right now? (You
  might have to search how to find this for your particular OS.)

* If a IP address is listed as 10.37.129.212/17, how many bits are used
  to represent the hosts?
