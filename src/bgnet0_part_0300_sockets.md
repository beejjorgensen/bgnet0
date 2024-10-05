# Introducing The Sockets API

In Unix, the sockets API generally gives processes a way to communicate
with one another. It supports a variety of methods of communication, and
one of those methods is over the Internet.

And that's the one we're interested in right now.

In C and Unix, the sockets API is a blend of library calls and system
calls (functions that call the OS directly).

In Python, the Python sockets API is a library that calls the
lower-level C sockets API. At least on Unix-likes. On other platforms,
it will call whatever API that OS exposes for network communication.

We're going to use this to write programs that communicate over the
Internet!

## Client Connection Process

The most confusing thing about using sockets is that there are generally
several steps you have to take to connect to another computer, and
they're not obvious.

But they are:

1. **Ask the OS for a socket**. In C, this is just a file descriptor (an
   integer) that will be used from here on to refer to this network
   connection. Python will return an object representing the socket.
   Other language APIs might return different things.

   But the important thing about this step is that you have a way to
   refer to this socket for upcoming data transmission. Note that it's
   not connected to anything yet at all.

2. **Perform a DNS lookup** to convert the human-readable name (like
   `example.com`) into an IP address (like 198.51.100.12).  DNS is the
   distributed database that holds this mapping, and we query it to get
   the IP address.

   We need the IP address so that we know the machine to connect to.

   Python Hint: While you can do DNS lookups in Python with
   `socket.getaddrinfo()`, just calling `socket.connect()` with a
   hostname will do the DNS lookup for you. So you can skip this step.

   Optional C Hint: Use `getaddrinfo()` to perform this lookup.

3. **Connect the socket** to that IP address on a specific port.

   Think of a port number like an open door that you can connect
   through. They're integers that range from 0 to 65535.

   A good example port to remember is 80, which is the standard port
   used for servers that speak the HTTP protocol (unencrypted).

   There must be a server listening on that port on that remote
   computer, or the connection will fail.

4. **Send data and receive data**. This is the part we've been waiting
   for.

   Data is sent as a sequence of bytes.

5. Close the connection. When we're done, we close the socket indicating
   to the remote side that we have nothing more to say. The remote side
   can also close the connection any time it wishes.

## Server Listening Process

Writing a server program is a little bit different.

1. **Ask the OS for a socket**. Just like with the client.

2. **Bind the socket to a port**. This is where you assign a port number
   to the server that other clients can connect to. "I'm going to be
   listening on port 80!" for instance.

   Caveat: programs that aren't run as root/administrator can't bind to
   ports under 1024--those are reserved. Choose a big, uncommon port
   number for your servers, like something in the 15,000-30,000 range.
   If you try to bind to a port another server is using, you'll get an
   "Address already in use" error.

   Ports are per-computer. It's OK if two different computers use the
   same port. But two programs on the same computer cannot use the same
   port on that computer.

   Fun fact: clients are bound to a port, as well. If you don't
   explicitly bind them, they get assigned an unused port when the
   connect--which is usually what we want.

3. **Listen for incoming connections**. We have to let the OS know
   when it gets an incoming connection request on the port we selected.

4. **Accept incoming connections**. The server will block (it will
   sleep) when you try to accept a new connection if none are pending.
   Then it wakes up when someone tries to connect.

   Accept returns a new socket! This is confusing. The original socket
   the server made in step 1 is still there listening for new
   connections. When the connection arrives, the OS makes a new socket
   _specifically for that one connection_. This way the server can
   handle multiple clients at once.

   Sometimes the server spawns a new thread or process to handle each
   new client. But there's no law that says it has to.

5. **Send data and receive data**. This is typically where the server
   would receive a request from the client, and the server would send
   back the response to that request.

6. **Go back and accept another connection**. Servers tend to be
   long-running processes and handle many requests over their lifetimes.

## Reflect

* What role does `bind()` play on the server side?

* Would a client ever call `bind()`? (Might have to search this one
  on the Internet.)

* Speculate on why `accept()` returns a new socket as opposed to just
  reusing the one we called `listen()` with.

* What would happen if the server didn't loop to another `accept()`
  call? What would happen when a second client tried to connect?

* If one computer is using TCP port 3490, can another computer use port
  3490?

* Speculate about why ports exist. What functionality do they make
  possible that plain IP addresses do not?

## Resources

* [Python Sockets Documentation](https://docs.python.org/3/library/socket.html)
* [flbg[_Beej's Guide to Network Programming_|bgnet]]--optional, for C devs
