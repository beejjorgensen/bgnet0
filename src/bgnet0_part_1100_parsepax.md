# Parsing Packets {#parsing-packets}

We've already seen some issues with receiving structured data from a
server. You call `recv(4096)`, and you only get 20 bytes back. Or you
call `recv(4096)` and it turns out the data is longer than that, and you
need to call it again.

There's an even worse issue there, too. If the server is sending you
multiple pieces of data, you might receive the first _and part of the
next_. You'll have a complete packet and the next partially complete
one! How do you reconstruct this?

An analogy might be if I needed you to split up individual sentences
from a block of text I give you, but you can only get 20 characters at a
time.

You call `recv(20)` and you get:

``` {.default}
This is a test of th
```

That's not a full sentence, so you can't print it yet. So you call
`recv(20)` again:

``` {.default}
This is a test of the emergency broadcas
```

Still not a sentence. Call it again:

``` {.default}
This is a test of the emergency broadcast system. This is on
```

Hey! There's a period in there, so we have a complete sentence. So we
can print it out. But we also have part of the next sentence already
received!

How are we going to handle all this in a graceful way?

## You Know What Would Make This Easy?

You know what would make this easy? If we abstracted it out and then we
could do something like this:

``` {.py}
while the connection isn't closed:
    sentence = get_next_packet()
    print(sentence)
```

Isn't that easier to think about? Once we have that code the extracts
the next complete packet from the data stream, we can just use it.

And if that code is complex enough, it could actually extract different
types of packets from the stream:

``` {.py}
packet = get_next_packet()

if packet.type == PLAYER_POSITION:
    set_player_position(packet.player_index, packet.player_position)

elif packet.type == PRIVATE_CHAT:
    display_chat(packet.player_from, packet.message, private=True)
```

and so on.

Makes things soooo much easier than trying to reason about packets as
collections of bytes that might or might not be complete.

Of course, doing that processing is the real trick. Let's talk about how
to make it happen.

## Processing a Stream into Packets

The big secret to making this work is this: make a big global buffer.

> A buffer is just another word for a storage area for a bunch of bytes.
> In Python, it would be a bytestring, which is convenient since you're
> already getting those back from `recv()`.

This buffer will hold the bytes you've seen so far. You will inspect the
buffer to see if it holds a complete data packet.

If there is a complete packet in there, you'll return it (as a
bytestring or processed). And also, critically, you'll strip it off the
front of the buffer.

Otherwise, you'll call `recv()` again to try to fill up the buffer until
you have a complete packet.

In Python, remember to use the `global` keyword to access global
variables, e.g.

``` {.py}
packet_buffer = b''

def get_next_packet(s):
    global packet_buffer

    # Now we can use the global version in here
```

Otherwise Python will just make another local variable that shadows the
global one.

## The Sentences Example Again

Let's look at that sentences example from the beginning of this
chapter.

We'll call our `get_sentence()` function, and it'll look at all the data
received so far and see if there's a period in it.

So far we have:

``` {.default}
  
```

Nothing. No data is received. There's no period in there so we don't
have a sentence, so we have to call `recv(20)` again to get more bytes:

``` {.default}
This is a test of th
```

Still no period. Call `recv(20)` again:

``` {.default}
This is a test of the emergency broadcas
```

Still no period. Call `recv(20)` again:

``` {.default}
This is a test of the emergency broadcast system. This is on
```

There's one! So we do two things:

1. Copy the sentence out so we can return it, and:

2. Strip the sentence from the buffer.

After step two, the first sentence is gone and the buffer looks like
this:

``` {.default}
This is on
```

and we return the first sentence "This is a test of the emergency
broadcast system."

And the function that called `get_sentence()` can print it.

And then call `get_sentence()` again!

In `get_sentence()`, we look at the buffer again. (Remember, the buffer
is global so it still has the data in it from the last call.)

``` {.default}
This is on
```

There's no period, so we call `recv(20)` again, but this time we only
get 10 bytes back:

``` {.default}
This is only a test.
```

But it's a complete sentence, so we strip it from the buffer, leaving it
empty, and then return it to the caller for printing.

### What If You Receive Multiple Sentences at Once?

What if I call `recv(20)` and get this back:

``` {.default}
Part 1. Part 2. Part
```

Well, it still works! The `get_sentence()` function will see the first
period in there, strip off the first sentence from the buffer so it
contains:

``` {.default}
Part 2. Part
```

and then return `Part 1.`.

The next time you call `get_sentence()`, as always, the first thing it
does is check to see if the buffer contains a full sentence. It does! So
we strip it off:

``` {.default}
Part
```

and return `Part 2.`

The next time you call `get_sentence()`, it sees no period in the
buffer, so there is no complete sentence, so it calls `recv(20)` again
to get more data.

``` {.default}
Part 3. Part 4. Part 5.
```

And now we have a complete sentence, so we strip it off the front:

``` {.default}
Part 4. Part 5.
```

and return `Part 3` to the caller. And so on.

## The Grand Scheme

Overall, you could think of this abstraction as a pipe full of data.
When there is a complete packet in the pipe, it's pulled off the front
and returned.

But if there's not, the pipe receives more data at the back and keeps
checking to see if it has an entire packet yet.

Here's some pseudocode:

``` {.py}
global buffer = b''    # Empty bytestring

function get_packet():
    while True:
        if buffer starts with a complete packet
            extract the packet data
            strip the packet data off the front of the buffer
            return the packet data

        receive more data

        if amount of data received is zero bytes
            return connection closed indicator

        append received data onto the buffer
```

In Python, you can slice off the buffer to get rid of the packet data
from the front.

For example, if you know the packet data is 12 bytes, you can slice it
off with:

``` {.py}
packet = buffer[:12]   # Grab the packet
buffer = buffer[12:]   # Slice it off the front
```

## Reflect

* Describe the advantages from a programming perspective to abstracting
  packets out of a stream of data.

