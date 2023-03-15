# Firewalls

A favorite component of every bad hacker movie ever is the _firewall_.
It's clear from those bad movies that it has something to do with
security, but the exact role is ambiguous; it's only apparent that a bad
guy getting through the firewall is a bad thing.

But back to reality: a firewall is a computer (often a router) that
restricts certain types of traffic between one interface and another. So
instead of forwarding every packet of every type from every port, it
might decide to drop those packets instead.

The upshot is that if someone in trying to connect to a port on the far
side of a firewall, the firewall might prevent that connection depending
on how it is configured.

In this chapter, we'll be speaking conceptually about firewalls, but
won't get into any specifics about practical configuration. There are
many implementations of firewalls out there, and they all tend to have
different methods of configuration.

## Firewall Operation

If you think about a packet arriving at a router, that router has the
capability to inspect that packet all it wants. It can look at the
source and destination IPs, or the ports, or even the application layer
data.

So that gives it the opportunity to make a decision about whether or not
to keep forwarding that packet based on any of that data.

For example, the firewall might be configured to allow any port 80
(HTTP) traffic to come from the outside of the firewall to the inside,
but block incoming traffic on all other destination ports.

Or maybe it'll be configured to only allow certain IP addresses to
connect.

So the firewall, on receipt of a packet, looks at its configured rules
and decides whether or not to drop the packet (if it's being filtered
out), or forward the packet normally.

When it decides to drop the packet, it has a more options. It can either
silently drop it, or it can reply with a
[ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol)
"destination unreachable" message.

That basic filtering gives some a control, but there are still
some things you might want to do that it can't manage.

For example, if someone behind the firewall establishes a TCP connection
over the Internet from a random port, we want to be able to get traffic
back to that host from that random port.

With plain filtering, we'd just have to allow traffic on all those
ports.

But with _stateful filtering_, the firewall tracks the state of TCP
connections that have been initiated from behind the firewall. The
firewall sees the TCP three-way handshake and knows then that there is a
valid connection. It can then safely allow traffic back that's destined
for the inside computer's random port number.

Stateful filtering is also used with UDP traffic, even though it's
connectionless. The firewall sees a UDP packet go out, and it'll allow
incoming UDP packets to that port. (For a while. Since it's
connectionless, there's no way to tell when the server and client and
done. So the firewall will timeout UDP rules after they haven't been
used for a while.)

> In order to keep the firewall from timing-out on any of its rules, the
> programs can send _keepalive_ messages. TCP actually has a specific
> message type for this, and the OS will periodically send empty `ACK`
> packets out to keep anyone from timing out. (If programming with
> sockets, you will likely need to set the `SO_KEEPALIVE` option.)
>
> The keepalive can also be effectively implemented by a program using
> UDP, as well. It could be configured to send a custom keepalive packet
> to the receiver (who would reply with some kind of ACK) every so
> often.
>
> Keepalive is only an issue for programs that have long periods of no
> network traffic.

Finally, filtering could also be done at the application layer. If the
firewall digs deeply enough into the packet, it might see that it's HTTP
or FTP data, for example. With that knowledge, it could allow or deny
all HTTP traffic, even if it's arriving on a non-standard port.

## Firewalls and NAT

In a home network, it's really common for the firewall to also be the
router and the switch and do NAT.

It's all rolled into one.

But there's nothing stopping us from having a separate firewall computer
on the network that only decides whether or not to allow traffic.

And there's no rule that one side of the firewall must be a private
network and the other side a public network. Both sides could be private
networks or public networks, or one private and one public.

The role of the firewall at its core is not to separate public and
private networks--that's NAT's role--but rather to control which traffic
is allowed to pass through the firewall.

## Local Firewalls

You can also set up a firewall just on your computer (and this is
commonly done). Typically this is set up to allow all connections from
the computer going out, but to restrict connections from other computers
coming in.

In MacOS and Windows, the firewall is something that can simply be
turned on and then it will start blocking traffic based on some common
rules.

If you need some specific ports unblocked, you'll have to manually add
those.

Linux has firewall support through a mechanism called
[iptables](https://en.wikipedia.org/wiki/Iptables). This isn't the most
straightforward thing to configure, but it's powerful enough to build a
NAT router/firewall from scratch.

## Reflect

* What's the difference/relationship between a firewall and a router?

* What's the difference/relationship between a firewall and NAT?

* What's the funniest or most painful thing in [this NCIS
  clip](https://www.youtube.com/watch?v=u8qgehH3kEQ)?

