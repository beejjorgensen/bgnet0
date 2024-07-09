# The Internet Protocol version 6

This is the new big thing! Since there are so few addresses
representable in 32 bits (only 4,294,967,296 not counting the reserved
ones), the Powers That Be decided we needed a new addressing scheme. One
with more bits. One that could last, for all intents and purposes,
forever.

There was a problem: we were running out of IP addresses. Back in the
1970s, a world with billions of computers was beyond imagination. But
today, we've already exceeded this by orders of magnitude.

So they decided to increase the size of IP addresses from 32 bits to 128
bits, which gives us 79,228,162,514,264,337,593,543,950,336 times as
much address space. This should genuinely last a looooooong time.

> Lots of this address space is reserved, so there aren't really that
> many addresses. But there are still a **LOT**, both imperial and
> metric.

That's the main difference between IPv4 and IPv6.

For demonstration purposes, we'll stick with IPv4 because it's still
common and a little easier to write out. But this is good background
information to know, since someday IPv6 will be the only game in town.

Someday.

## Representation

With that much address space, dots-and-decimal numbers won't cut it.  So
they came up with a new way of displaying IPv6 addresses: colons-and-hex
numbers. And each hex number is 16 bits (4 hex digits), so we need 8 of
those numbers to get us to 128 bits.

For example:

``` {.default}
2001:0db8:6ffa:8939:163b:4cab:98bf:070a
```

Slash notation is used for subnetting just like IPv4. Here's an example
with 64 bits for network (as specified with `/64`) and 64 bits for host
(since `128-64=64`):

``` {.default}
2001:0db8:6ffa:8939:163b:4cab:98bf:070a/64
```

64 bits for host! That means this subnet can have
18,446,744,073,709,551,616 hosts!

There's a lot of space in an IPv6 address!

> When we're talking about standard IPv6 addresses for particular hosts,
> `/64` is the strongly-suggested rule for how big your subnet is. Some
> protocols rely on it.
> 
> But when we're just talking about subnets, you might see smaller
> numbers there representing larger address spaces. But the expectation
> is that eventually that space will be partitioned down into `/64`
> subnets for use by individual hosts.

Now, writing all those hex numbers can be unwieldy, especially if there
are large runs of zeros in them. So there are a couple shortcut rules.

1. Leading zeros on any 16-bit number can be removed.
2. Runs of multiple zeros after rule 1 has been applied can be replaced
   with two sequential colons.

For example, we might have the address:

``` {.default}
2001:0db8:6ffa:0000:0000:00ab:98bf:070a
```

And we apply the first rule and get rid of leading zeros:

``` {.default}
2001:db8:6ffa:0:0:ab:98bf:70a
```

And we see we have a run of two `0`s in the middle, and we can replace
that with two colons:

``` {.default}
2001:db8:6ffa::ab:98bf:70a
```

In this way we can get a more compact representation.

## Link-Local Addresses

[This is "good to know" information, but just file it away under "IPv6
automatically gives all interfaces an IP address".]

There are addresses in IPv6 and IPv4 that are reserved for hosts on this
particular LAN. These aren't commonly used in IPv4, but they're required
in IPv6. The addresses are all on subnet `fe80::/10`.

> Expanded out, this is:
> ``` {.default}
> fe80:0000:0000:0000:0000:0000:0000:0000
> ```

The first 10 bits being the network portion. In an IPv6 link-local
address, the next 54 bits are reserved (`0`) and then there are 64 bits
remaining to identify the host.

When an IPv6 interface is brought up, it automatically computes its
link-local address based on its Ethernet address and other things.

Link-local addresses are unique on the LAN, but might not be
globally-unique. Routers do not forward any link-local packets out of
the LAN to prevent issues with duplicate IPs.

An interface might get a different IP later if a DHCP server hands one
out, for example, in which case it'll have two IPs.

## Special IPv6 Addresses and Subnets

Like with IPv4, there are a lot of addresses that have special meaning.

* **`::1`** - localhost, this computer, IPv6 version of `127.0.0.1`
* **`2001:db8::/32`** - for use in documentation
* **`fe80::/10`** - link local address

There are IPv6 ranges with special meanings, but those are the common
ones you'll see.

## IPv6 and DNS

DNS maps human-readable names to IPv6 addresses, as well. You can look
them up with `dig` by telling it to look for an `AAAA` record (which is
what DNS calls IPv6 address records).

``` {.default}
$ dig example.com AAAA
```

``` {.default}
; <<>> DiG 9.10.6 <<>> example.com AAAA
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13491
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;example.com.			IN	AAAA

;; ANSWER SECTION:
example.com.		81016	IN	AAAA	2606:2800:220:1:248:1893:25c8:1946

;; Query time: 14 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Wed Sep 28 16:05:16 PDT 2022
;; MSG SIZE  rcvd: 68
```

You can see the IPv6 address of `example.com` in the `ANSWER SECTION`,
above.

## IPv6 and URLs

Since a URL uses the `:` character to delimit a port number, that
meaning collides with the `:` characters used in an IPv6 address.

If you run your server on port 33490, you can connect to it in your web
browser by putting the IPv6 address in square brackets. For example, to
connect to localhost on address `::1`, you can:

``` {.default}
http://[::1]:33490/
```

## Reflect

* What are some benefits of IPv6 over IPv4?

* How can the address `2001:0db8:004a:0000:0000:00ab:ab4d:000a` be
  written more simply?
