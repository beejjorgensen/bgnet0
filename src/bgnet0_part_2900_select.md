# Select

In this chapter we're taking a look at the `select()` function. This is
a function that looks at a whole set of sockets and lets you know which
ones have sent you data. That is, which ones are ready to call `recv()`
on.

This enables us to wait for data on a large number of sockets at the
same time.

## The Problem We're Solving

Let's say the server is connected to three clients. It wants to `recv()`
data from whichever client sends it next.

But the server has no way of knowing which client will send data next.

In addition, when the server calls `recv()` on a socket with no data
ready to read, the `recv()` call _blocks_, preventing anything else from
running.

> To _block_ means that the process will stop executing here and goes to
> sleep until some condition is met. In the case of `recv()`, the
> process goes to sleep until there's some data to receive.

So we have this problem where if we do something like this:

``` {.py}
data1 = s1.recv(4096)
data2 = s2.recv(4096)
data3 = s3.recv(4096)
```

but there's no data ready on `s1`, then the process will block there and
not call `recv()` on `s2` or `s3` even if there's data to be received on
those sockets.

We need a way to monitor `s1`, `s2`, and `s3` at the same time,
determine which of them have data ready to receive, and then call
`recv()` only on those sockets.

The `select()` function does this. Calling `select()` on a set of
sockets will block until one or more of those sockets is ready-to-read.
And then it returns to you which sockets are ready and you can call
`recv()` specifically on those.

## Using `select()`

First of all, you need the `select` module.

``` {.py}
import select
```

If you have a bunch of connected sockets you want to test for being
ready to `recv()`, you can add those to a `set()` and pass that to
`select()`. It will block until one is ready to read.

This set can be used as your canonical list of connected sockets. You
need to keep track of them all somewhere, and this set is a good place.
As you get new connections, you add them to the set, and as the
connections hang up, you remove them from the set. In this way, it
always holds the sockets of all the current connections.

Here's an example. `select()` takes three arguments and return three
values. We'll just look at the first of each of these for now, ignoring
the other ones.

``` {.py}
read_set = {s1, s2, s3}

ready_to_read, _, _ = select.select(ready_set, {}, {})
```

At this point, we can go through the sockets that are ready and receive
data.

``` {.py}
for s in ready_to_read:
    data = s.recv(4096)
```

## Using `select()` with Listening Sockets

If you've been looking closely, you might have the following question:
if the server is blocked on a `select()` call waiting for incoming data,
how can it also call `accept()` to accept incoming connections? Won't
the incoming connections have to wait? Furthermore, `accept()` blocks...
how will we get back to the `select()` if we're blocked on that?

Fortunately, `select()` provides us with an answer: _you can add a
listening socked to the set!_ When the listening socket shows up as
"ready-to-read", it means there's a new incoming connection to
`accept()`.

## The Main Algorithm

Putting it all together, we get the core of any main loop that uses
`select()`:

``` {.default}
add the listener socket to the set

main loop:

    call select() and get the sockets that are ready to read

    for all sockets that are ready to read:

        if the socket is the listener socket:
            accept() a new connection
            add the new socket to our set!

        else the socket is a regular socket:
            recv() the data from the socket

            if you receive zero bytes
                the client hung up
                remove the socket from tbe set!
```

## What About Those Other Arguments to `select()`?

`select()` actually takes three arguments. (Though for this project we
only need to use the first one, so this section is purely informational.)

They correspond to:

* Which sockets you want to monitor for ready-to-read
* Which sockets you want to monitor for ready-to-write
* Which sockets you want to monitor for exception conditions

And the return values map to these, as well.

``` {.py}
read, write, exc = select.select(read_set, write_set, exc_set)
```

But again, for this project, we just use the first and ignore the rest.

``` {.py}
read, _, _ = select.select(read_set, {}, {})
```

### The Timeout

I told a bit of a lie. There's an optional fourth arguments, the
`timeout`. It's a floating point number of seconds to wait for an event
to occur; if nothing occurs in that timeframe, `select()` returns and
none of the returned sockets are shown as ready.

You can also specify a timeout of `0` if you want to just poll the
sockets.

## Reflect

* Why can't we just call `recv()` on all the connected sockets? What
  does `select()` buy us?

* When `select()` shows a socket "ready-to-read", what does it mean if
  the socket is a listening socket versus a non-listening socket?

* Why do we have to add the listener socket to the set, anyway? Why not
  just call `accept()` and then call `select()`?

