# Network Address Translation (NAT)

In this chapter we'll be taking a look at _network address translation_,
or NAT.

This is a service provided on a router which hides a "private" LAN from
the rest of the world.

The router acts as a middleman, translating internal IP addresses to a
single external IP address. It keeps a table of these connections so
when packets arrive on either interface, the router can rewrite them
appropriately.

We say the LAN behind the NAT router is a "private network", and the
external IP address the router presents for that LAN is the "public IP".

## A Snail-Mail Analogy

Imagine how an anonymous snail-mail program might work.

You write a letter addressed to the recipient with your return address
on it.

Instead of mailing it directly, you hand the letter to an anonymizer.
The anonymizer removes your letter from the envelope and puts it in
another envelope with the same recipient, but replaces the return
address with the anonymnizer's address. It also records that this sender
wrote a letter to this recipient.

The recipient gets the letter. All they see is the anonymizer's return
address on the letter.

The recipient responds to the letter, and sends the response back to the
anonymizer. The recipient lists themselves as the return address.

The anonymizer receives the mail and notes it's from the recipient. It
looks in its records to determine the person who originally sent a
message to the recipient.

The anonymizer takes the response out of the envelope and puts it in a
new envelope with the destination address being the original sender, and
the return address remaining the original recipient.

The original sender receives the response.

Note that the original recipient never knew the original sender's
address; they only knew the anonymizer's address.

## Why?

NAT is very, very common. It almost certainly runs on every IPv4 LAN.

But why?

There are two main reasons to use this type of service.

1. You want to hide your network details from the rest of the world.

2. You need more IP addresses than either (a) anyone is willing to
   allocate to you or (b) you're willing to pay for.

### Hiding Your Network

There's no easy way to get random, unsolicited packets onto the private
network through the gateway running NAT.

> You can possibly do it with something called _source routing_, but
> that's not commonly enabled by ISPs.

You'd have to have the private IP on the packet **and** get that packet
to the router. There's no way to do this unless you're on the same LAN
as the router, and that's really unlikely.

Regarding the second point, we should talk about the phenomenon known as
_address space exhaustion_.

### IPv4 Exhaustion

There are really a limited number of IPv4 addresses in the world. As
such, getting a large block of them costs a lot of money.

It's a lot cheaper to get just a handful of addresses (e.g. a `/20` or
`/24` or `/4` network) and then have large private networks behind NAT
routers. This way you can have a _lot_ of IPs, but they all present to
the world as coming from the single public IP.

This is really the motivation behind using NAT everywhere. There was a
time when we were genuinely about to run out of IPv4 addresses, and NAT
saved us.

## Private Networks

There are subnets that are reserved for use as private networks. Routers
don't forward these addresses directly. If a router on the greater
Internet sees an IP from one of these subnets, it will drop it.

For IPv4 there are three such subnets:

* `10.0.0.0/8`
* `172.16.0.0/12`
* `192.168.0.0/16`

The last of these is really common on household LANs.

Again, routers on the Internet will drop packets with these addresses.
The only way data gets from these IPs onto the Internet is via NAT.

## How it Works

For this demo, we're going to use `IP:port` notation. The number after
the colon is the port number.

Also, we'll use the term "Local Computer" to refer to the computer on
the LAN, and "Remote Computer" or "Remote Server" to refer to the
distant computer it's connecting to.

And finally, the local LAN router will just be called "The Router" or
the "NAT Router".

Let's go!

On my LAN, I want to go from `192.168.1.2:1234` on my private LAN to the
public address `203.0.113.24:80`--that's port `80`, the HTTP server.

The first thing my computer does is check to see if the destination IP
address is on my LAN... Since my LAN is `192.168.0.0/16` or smaller,
then no, it's not.

So my computer sends it to the default gateway, my router that has NAT
enabled.

The router is going to play the role of the "anonymizer" middleman in
the earlier analogy example. And recall that the router has two
interfaces on it--one faces the internal `192.168.0.0/16` private LAN,
and the other faces the greater Internet with a public, external IP.
Let's use `192.168.1.1` as the private IP address on the router and
`198.51.100.99` as the public IP.

So the LAN looks like this:

``` {.default}
+----------+-------------+
|  Local   |             |
| Computer | 192.168.1.2 |>-------+
+----------+-------------+        |
                                  |
                                  ^
                          +---------------+
                          |  192.168.1.1  |  Internal IP
                          +---------------+
                          |    Router     |  NAT-enabled
                          +---------------+
                          | 198.51.100.99 |  External IP
                          +---------------+
                                  v
                                  |
                                  |
                       {The Greater Internet}
```

The router now has to "repackage" the data for the greater Internet.
This means rewriting the packet source to be from the router's public IP
and some unused port on the public interface of the router. (The port
doesn't need to be the same as the port originally on the packet. The
router allocates a random unused port when the new packet is sent out.))

So the router records all this information:

<!-- CAPTION: NAT router connection record -->

|Computer|IP|Port|Protocol|
|-|-|-|-|
|Local Private Address|192.168.1.2|1234|TCP|
|Remote Public Address|203.0.113.24|80|TCP|
|Router Public Address|198.51.100.99|5678|TCP|

(Note: except for `80`, since we're connecting to HTTP in this example,
all port numbers were randomly chosen.)

So while the original packet was:

``` {.default}
192.168.1.2:1234 --> 203.0.113.24:80

     Local               Remote
    Computer            Computer
```

the NAT router rewrites it to be the same destination, but the router as
the source.

``` {.default}
198.51.100.99:5678 --> 203.0.113.24:80

    NAT router             Remote
                          Computer
```

From the destination's point of view, it is completely unaware that this
isn't originally from the router itself.

So the destination replies with some HTTP data, sending it back to the
router:

``` {.default}
203.0.113.24:80 --> 198.51.100.99:5678

    Remote              NAT router
   Computer
```

The router then looks at its records. It says, "Wait a moment--if I get
data on port `5678`, that means I need to translate this back to a
private IP on the LAN!

So it translates the message so that it's no longer addressed to the
router, but instead is sent to the private source IP recorded earlier:

``` {.default}
203.0.113.24:80 --> 192.168.1.2:1234

    Remote               Local
   Computer             Computer
```

And sends that out on the LAN. And its received! The LAN computer thinks
it's talking to the remote server, and the remote server thinks it's
talking to the router! NAT!

It might be a little confusing in that last step that the packet is
coming from the NAT router but is actually IP addressed like it came
from the Remote Computer. This is OK on the LAN because the NAT router
sends that IP packet out with an Ethernet address of the Local Computer
on the LAN. Or, put another way, the NAT router can use link layer
addressing to get the packet delivered to the Local Computer and the
IP address of where that packet came from doesn't have to match the
internet IP of the NAT router.

## NAT and IPv6

Since IPv6 has _tons_ of addresses, do we really need NAT? And the
technical answer is "no". Part of the reason IPv6 came about was to get
rid of this nasty NAT middleman business.

That said, there is a reserved IPv6 subnet for private networks:

``` {.default}
fd00::/8
```

There's some additional structure in the "host" portion of the address,
but you can [read about it on
Wikipedia](https://en.wikipedia.org/wiki/Unique_local_address#Definition)
if you want.

Like the IPv4 private subnets, IP addresses from this subnet are dropped
by routers on the greater Internet.

NAT doesn't work for IPv6, but there is another way to do the
translation with [network prefix
translation](https://en.wikipedia.org/wiki/IPv6-to-IPv6_Network_Prefix_Translation).

But what about the whole idea that people can't easily get unsolicited packets onto
your LAN? Well, you're just going to have to configure your firewall
properly to keep people out. But that's a story for another time.

## Port Forwarding

If you have a server running behind the NAT router, how can it serve
content to the outside world? After all, no one on the outside can refer
to it by its internal IP address--all they can see is the router's
external IP.

But you can configure the NAT router to do something called _port
forwarding_. 

For example, you could tell it that traffic sent to its public IP port
80 should be _forwarded_ to some private IP on port 80. The router
forwards the traffic, and the original sender is unaware that its
traffic is ultimately arriving at a private IP.

There's no reason the same port must be used.

For example, SSH uses port 22, and that's fine on the computer on the
private network. But if you forward from the public port 22, you'll find
malicious actors are continuously trying to log in through it. (Yes,
even on your computer at home.) So it's more common to use some other
uncommon port as the public SSH port, and then forward it to port 22 on
the LAN.

## Review

* Look up your computer's internal IP address. (Might have to search the
  net to see how to do this.) Then go to
  [google.com](https://google.com/) and type "what is my ip". The
  numbers are (very probably) different. Why?

* What problems does NAT solve?

