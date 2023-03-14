# Project: Multiuser Chat Client and Server

It's time to put it all together into this, the final project!

We're going to build a multiuser chat server and a chat client to go
along with it.

The chat server should allow an arbitrary number of connections from
clients. All the clients will see what the other ones are saying.

Not only that, but there should be messages for when a user joins or
leaves the chat.

Here's a sample screenshot. The prompt where this user ("pat" in this
example) is typing what they're about to say. The region above it is
where all the output accumulates.

```
*** pat has joined the chat
pat: Hello?
*** leslie has joined the chat
leslie: hi everyone!
pat: hows it going
*** chris has joined the chat
chris: OK, now we can start!
pat: lol
leslie: Why are you always last?
*** chris has left the chat

pat> oh no!! :)█
```

## Overall Architecture

There will be one server, and it will handle many simultaneous clients.

### Server

The server will run using `select()` to handle multiple connections to
see which ones are ready to read.

The listener socket itself will also be included in this set. When it
shows "ready-to-read", it means there's a new connection to be
`accept()`ed. If any of the other already-accepted sockets show
"ready-to-read", it means the client has sent some data that needs to be
handled.

When the server get a chat packet from one client, it rebroadcasts that
chat message to every connected client.

> Note: when we use the term "broadcast" here, we're using it in the
> generic sense of sending a thing to a lot of people. We're **not**
> talking about IP or Ethernet broadcast addresses. We won't use those
> in this project.

When a new client connects or disconnects, the server broadcasts that
to all the clients, as well.

Since multiple clients will be sending data streams to the server, the
server needs to maintain a packet buffer _for each client_.

> You can put this is a Python `dict` that uses the client's socket
> itself as the key, so it maps from a socket to a buffer.

The server will be launched by specifying a port number on the command
line. This is mandatory; there is no default port.

```
python chat_server.py 3490
```

### Client

When the client is launched, the user specifies their nickname (AKA
"nick") on the command line along with the server information.

The very first packet it sends is a "hello" packet that has the nickname
in it. (This is how the server associates a connection with a nick and
rebroadcasts the connection event to the other clients.)

After that, every line the user types into the client gets sent to the
server as a chat packet.

Every chat packet (or connect or disconnect packet) the client gets from
the server is shown on the output.

The client has a **text user interface** (TUI) that helps keep the
output clean. Since the output is happening asynchronously on a
different part of the screen than the input, we need to do some terminal
magic to keep them from overwriting each other. This TUI code will be
supplied and is described in the next section.

Since there can be data arriving while the user is typing something, we
need a way to handle that. The client will be **multithreaded**. There
will be two threads of execution.

* The main sending thread will:
  * Read keyboard input
  * Send chat messages from the user to the server
* The receiving thread will:
  * Receive packets from the server
  * Display those results on-screen

Since there is no shared data between those threads, no synchronization
(mutexes, etc.) will be required.

> They do share the socket, but the OS makes sure that it's OK for
> multiple threads to use that at the same time without an issue. It's
> _threadsafe_.

If you need more information on threading in Python, see the [Appendix:
Threading](#appendix-threading) section.

The client will be started by specifying the user's nickname, the server
address, and the server port on the command line. These are all required
arguments; there are no defaults.

```
python chat_client.py chris localhost 3490
```

## Client I/O

The client screen is split into two main regions:

* The input section, the last row or two of the screen.
* The output section, the remaining top part of the screen.

(The Client TUI section, below, has details about how to do this I/O.)

The client input line at the bottom of the screen should be the user's
nickname followed by `>` and a space. The input takes place after that:

```
alice> this is some sample input
```

The output area of the screen has two main types of messages:

* **Chat messages**: these show the speaker nickname followed by `:` and
  a space, and then the message.

  ```
  pat: hows it going
  ```

* **Informational messages**: these show when a user has joined or left
  the chat, or any other information that needs printing. They consist
  of `***` followed by a space, then the message. Joining and leaving
  messages are shown here:

  ```
  *** leslie has joined the chat
  *** chris has left the chat
  ```

### Special User Input

If the user input begins with `/`, it has special meaning and should be
parsed farther to determine the action.

Currently, the only special input defined is `/q`:

* **`/q`**: if the user enters this, the client should exit. Nothing is
  sent to the server in this case.

## The Client TUI

[fls[Download the Chat UI code and demo here|chat/chatui.zip]].

In the file `chatui.py`, there are four functions you need, and you can
get them with this import:

```
from chatui import init_windows, read_command, print_message, end_windows
```

The functions are:

* **init_windows()**: call this first before doing any other
  UI-oriented I/O of any kind. It should also be called before you start
  the receiver thread since that thread does I/O.

* **end_windows()**: call this when your program completes to clean
  everything up.

* **read_command()**: this prints a prompt out at the bottom of the
  screen and accepts user input. It returns the line the user entered
  once they hit the `ENTER` key.

  For example:

  ```
  s = read_command("Enter something> ")
  ```

  The function takes care of screen placement of the element.

* **print_message()**: Prints a message to the output portion of the
  screen. This handle scrolling and making sure the output doesn't
  interfere with the input from `read_command()`.

  Do not include a newline in your output. It will be added
  automatically.

**Known Bug**: on Mac, if something gets written by `print_message()`,
then next backspace you type will show a `^R` and scroll down a line.
It's unclear why this happens.

### Curses Variant of `chatui`

If the `chatui` library is causing you trouble, you could try the
alternate version `chatuicurses`. It has the exact same functions and is
used in the exact same way.

Before you use it, you have to install the unicurses library:

```
python3 -m pip install uni-curses
```

After that installs, you should just be able to use `chatuicurses`
instead of `chatui` in the import line.

**Known Mac Issue**: my attempt complained that the curses library
wasn't installed when in fact it was. This doesn't seem to affect Linux
or Windows.

**One caveat** here is that the input routine doesn't obey `CRTL-C` to
get out of the app. As such, you might have to hit `CTRL-C` followed by
`RETURN` to actually break out. On Windows, you might try `CTRL-BREAK`.

## Packet Structuree

The client and server will communicate over TCP (stream) sockets using a
defined packet structure.

In a nutshell, a packet is a 16-bit big-endian number representing the
payload length. The payload is a string containing JSON-format data with
UTF-8 encoding.

> You can encode the JSON string to UTF-8 bytes by calling `.encode()`
> on the string. `.encode()` takes an argument to specify the encoding,
> but it defaults to `"UTF-8"`.

So the first thing you'll have to do when looking at the data stream is
make sure you have at least two bytes in your buffer so you can
determine the JSON data length. And then after that, see if you have the
length (plus 2 for the 2-byte header) in your buffer.

## JSON Payloads

If your JSON is rusty, check out the [Appendix: JSON](#appendix-json)
section.

Each packet starts with the two-byte length of the payload, followed by
the payload.

The payload is a UTF-8 encoded string representing a JSON object.

Each payload is an Object, and has a field in it named `"type"` that
represents the type of the payload. The remaining fields vary based on
the type.

In the following examples, square brackets in strings are used to
indicate a place where you need to put in the relevant information. The
brackets are **not** to be included in the packet.

### "Hello" Payload

When a client first connects to a server, it sends a `hello` packet with
the user's nickname. This allows the server to associate a connection
with a nick.

This MUST be sent before any other packets.

From client to server:

```
{
    "type": "hello"
    "nick": "[user nickname]"
}
```

### "Chat" Payload

This represents a chat message. It has two forms depending on whether or
not the chat message originated with the client (i.e. the user wants to
send a message) or the server (i.e. the server is broadcasting someone
else's message).

From client to server:

```
{
    "type": "chat"
    "message": "[message]"
}
```

From server to clients:

```
{
    "type": "chat"
    "nick": "[sender nickname]"
    "message": "[message]"
}
```

The client doesn't need to send the sender's nick along with the packet
since the server can already make that association from the `hello`
packet sent earlier.

### "Join" Payload

The server sends this to all the clients when someone joins the chat.

```
{
    "type": "join"
    "nick": "[joiner's nickname]"
}
```

### "Leave" Payload

The server sends this to all the clients when someone leaves the chat.

```
{
    "type": "leave"
    "nick": "[leaver's nickname]"
}
```

## Extensions

These aren't worth any points, but if you want to push farther, here are
some ideas. **Caveat!** Be sure whatever you turn in has the official
functionality as already described. These mods can be a strict superset
of that, or you can fork a new project to hold them.

At the very least, I recommend branching from your working version so it
doesn't get accidentally messed up!

* Add direct messaging--if the user "pat" sends:
  ```
  /message chris how's it going?
  ```
  then "chris" will see:
  ```
  pat -> chris: how's it going?
  ```
  (What if the user doesn't exist? Maybe you need to define an error
  packet to get back from the server!)

* Add a way to list the names of all the people in the chat

* Add emotes--if the user "pat" sends:
  ```
  /me goes out to buy some snacky cakes
  ```
  everyone else sees:
  ```
  [pat goes out to buy some snacky cakes]
  ```
  (The right way to do this is to add a new packet type!)

* Add chat rooms--there could be a default room everyone is in when they
  first join, but additions could be to add chat rooms, join or leave
  chat rooms, and list available chat rooms.

* Turn the whole thing into a [MUD](https://en.wikipedia.org/wiki/MUD).
  That should keep you busy!

## Some Recommendations

Here are some pointers that might help.

  * Have the server use the set of connected sockets that it passes to
    `select()`  as its canonical list of everyone who is connected.

    If a client disconnects, remove them from the set.

    If a new client connects, add them to the set.

    The set will always reflect everyone who is connected at this time.

  * Have a buffer per connection on the server. Remember how we had a
    buffer in earlier projects that would accumulate data until there
    was a complete packet? We need one of those buffers _per
    connection_. And in this project, we have a lot of connections.

    Use a `dict` to store the buffers. The key is the socket itself, and
    the value is a string with the buffer for that socket in it.

  * Remember the project where we wrote code to extract packets from the
    bytestring buffer?

    Use that strategy again.

  * You'll want to use `.to_bytes()` and `.from_bytes()` to get and set
    the packet length.

  * Use old projects as much as you can.

<!-- Rubric

5 points each

Client sends correct `chat` packet JSON (nickname not included)

Client sends correct `hello` packet JSON

Client handles `join` packet JSON

Client handles `leave` packet JSON

Client implements the text user interface from `chatui`

Client is multithreaded, one thread for receiving and one for input and sending

Client extracts len+JSON packets from stream correctly

Client JSON strings are UTF-8 encoded

Client encodes packets correctly with the length and JSON payload

Client accepts nickname, server, and port number on the command line

Client `/q` command implemented correctly

Server sends correct `chat` packet JSON

Server sends correct `join` packet JSON

Server sends correct `leave` packet JSON

Server handles `hello` packet JSON

Server uses select to handle multiple connections

Server extracts len+JSON packets from stream correctly

Server JSON strings are UTF-8 encoded

Server encodes packets correctly with the length and JSON payload

Server accepts port number on the command line
-->

