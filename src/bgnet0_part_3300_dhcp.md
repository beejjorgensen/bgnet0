# Dynamic Host Configuration Protocol (DHCP)

When you first open your laptop at the coffee shop, it doesn't have an
IP address. It doesn't even know what IP address it _should_ have. Or
what it's name servers are. Or what its subnet mask is.

Of course, you could manually configure it! Just type in the numbers
that the cashier hands you with your coffee!

OK, that doesn't happen. No one would bother. Or they'd use a duplicate
address. And things wouldn't work. And they'd probably rage-drink their
coffee and never return.

It would be better if there were a way to automatically configure a
computer that just arrived on the network, wouldn't it?

That's what the _Dynamic Host Configuration Protocol_ (DHCP) is for.

## Operation

The overview:

``` {.default}
Client --> [DHCPDISCOVER packet] --> Server

Client <-- [DHCPOFFER packet] <-- Server

Client --> [DHCPREQUEST packet] --> Server

Client <-- [DHCPACK packet] <-- Server
```

The details:

When your laptop first tries to connect to the network, it sends a
`DHCPDISCOVER` packet to the broadcast address (`255.255.255.255`) over
UDP to port `67`, the DHCP server port.

Recall that the broadcast address only propagates on the LAN--the
default gateway does not forward it.

On the LAN is another computer acting as the DHCP server. There's a
process running on it waiting on port `67`.

The DHCP server process sees the DISCOVER and decides what to do with
it.

The typical use is that the client wants an IP address. We call this
_leasing_ an IP from the DHCP server. The DHCP server is tracking which
IPs have been allocated and which are free out of its pool.

In response to the `DHCPDISCOVER` packet, the DHCP server sends a
`DHCPOFFER` response back to the client on port `68`.

The offer contains an IP address and potentially a lot of other pieces
of information including, but not limited to:

* Subnet mask
* Default gateway address
* The lease time
* DNS servers

The client can accept or ignore the offer. (Maybe there are multiple
DHCP servers making offers, but the client may only accept one of them.)

If the offer is accepted, the client sends a `DHCPREQUEST` back to the
server notifying it that it wants that particular IP address.

Finally, if all's well, the server replies with an acknowledgment
packet, `DHCPACK`.

At that point, the client has all the information it needs to
participate on the network.

## Reflect

* Reflect on the advantages of using something like DHCP over manually
  configuring the devices on your LAN.

* What types of information does a DHCP client receive from the DHCP
  server?
