# Port Scanning

**A NOTE ON LEGALITY**: It's unclear whether or not it is legal to
portscan a computer you do not own. We'll be doing all our portscanning
on `localhost`. **Don't portscan computers you do not have permission
to!**

A port scan is a method of determining which ports on a computer a ready
to accept connections. In other words, which ports have a server
listening on them.

The port scan is often the first line of attack on a system. Before
being able to connect to see if some vulnerabilities can be found in any
listening servers, we need to know which servers are running in the
first place.

## Mechanics of a Portscanner

A portscanner will attempt to find listening server processes on a range
of ports on a specified IP. (Sometimes the range is all ports.)

With TCP, the general approach is to try to set up a connection with the
`connect()` call on the candidate port. If it works, we have an open
port, and the portscanner outputs that information. Nothing is sent over
the connection, and it is closed immediately.

> The job of the portscanner is to identify open ports, not to send or
> receive other data unnecessarily.

Calling `connect()` causes the TCP connection to complete its three-way
handshake. This isn't strictly necessary, since if the portscanner gets
any reply from the server (i.e. the second part of the three way
handshake) then we know the port is open. Completing the handshake would
cause the remote OS to wake up the server on that port for a connection
we know we're just going to close anyway.

Some TCP portscanners will instead build a TCP SYN packet to start the
handshake and send that to the port. If they get a SYN-ACK reply, they
know the port is open. But then, instead of completing the handshake
with an ACK, they send a RST (reset) causing the remote OS to terminate
the nascent connection entirely--and causing it to not wake up the
server on that port.

> You can write software that builds custom TCP packets with [raw
> sockets](https://en.wikipedia.org/wiki/Network_socket#Types). Usually
> you need to be a superuser/admin to use these.

### Portscanning UDP

Since there is no connection with UDP, port scans are a little less
exact.

One technique is to send a UDP packet to a port and see if you get an
[ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol)
"destination unreachable" message back. If so, the port is not open.

If you don't get a response, _maybe_ the port is open. Or maybe the UDP
packet is being filtered out and dropped with no ICMP response.

Another option is to send a UDP packet to a known port that might have a
server behind it. If you get a response from the server, you know the
port is open. If not, it's closed or your traffic was filtered out.

## Reflect

* Why shouldn't you portscan random computers on the Internet?

* What's the purpose of using a portscanner?
