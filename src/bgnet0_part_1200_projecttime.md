# Project: Atomic Time

You're going to reach out to the atomic clock at NIST (National
Institute of Standards and Technology) and get the number of seconds
since January 1, 1900 from their clocks. (That's a lot of seconds.)

And you'll print it out.

And then you're going to print out the system time from the clock on
your computer.

If your computer's clock is accurate, the numbers should be very close
in the output:

``` {.default}
NIST time  : 3874089043
System time: 3874089043
```

We're just writing a client in this case. The server already exists and
is already running.

## Note On Legality

The NIST runs this server for public use. Generally speaking, you don't
want to be connecting to servers where the owner doesn't want you to.
It's a quick way to run afoul of the law.

But in this case, the general public is welcome to use it.

## Note On Allowable Use

**The NIST server should never be queried more than once every four
seconds**. They might start refusing service if you exceed this rate.

If you're running the code frequently and want to be sure you've waited
4 seconds, you can buffer the run with a `sleep` call on the command
line:

``` {.sh}
sleep 4; python3 timeclient.py
```

## Epoch

In computer parlance, we refer to "epoch" as meaning "the beginning of
time" from a computer perspective.

Lots of libraries measure time in "number of seconds since epoch",
meaning since the dawn of time.

What do we mean by the dawn of time? Well, it's depends.

But in Unix-land, the dawn of time is very specifically January 1, 1970
at 00:00 UTC (AKA Greenwich Mean Time).

In other epochs, the dawn of time might be another date. For example,
the Time Protocol that we'll be speaking uses January 1, 1900 00:00 UTC,
70 years before Unix's.

This means we'll have to do some conversion. But luckily for you, we'll
just give you the code that will return the value for you and you don't
have to worry about it.

## A Dire Warning about Zeros

For reasons I don't understand, sometimes the NIST server will return 4
bytes of all zeros. And other times it will just send zero bytes and
close the connection.

If this happens, you'll probably see `0` show up as the NIST time.

Just try running your client again to see if you get good results after
one or two more tries, keeping in mind the restriction of one query
every 4 seconds.

They're on a rotating IP for `time.nist.gov` and it seems like one or
two of the servers might not be working right.

If it keeps coming up zero, something else might be wrong.

## The Gameplan

The is what we'll be doing:

1. Connect to the server `time.nist.gov` on port `37` (the Time Protocol
   port).

2. Receive data. (You don't need to send anything.) You should get 4
   bytes.

3. The 4 bytes represent a 4-byte big-endian number. Decode the 4 bytes
   with `.from_bytes()` into a numeric variable.

4. Print out the value from the time server, which should be the number
   of seconds since January 1, 1900 00:00.

5. Print the system time as number of seconds since January 1, 1900
   00:00.

The two times should loosely (or exactly) agree if your computer's clock
is accurate.

The number should be a bit over 3,870,000,000, to give you a ballpark
idea. And it should increment once per second.

### 1. Connect to the Server

The Time Protocol in general works with both UDP and TCP. For this
project you must use TCP sockets, just like we have been for other
projects.

So make a socket and connect to `time.nist.gov` on port `37`, the Time
Protocol port.

### 2. Receive the Data

Technically you should use a loop to do this, but it's very unlikely
that such a small amount of data will be split into multiple packets.

You'll receive 4 bytes max, no matter how many you ask for.

You can close the socket right after the data is received.

### 3. Decode the Data

The data is an integer encoded as 4 bytes, big-endian.

Use the `.from_bytes()` method mentioned in the earlier chapters to
convert the bytestream from `recv()` to a value.


### 4. Print Out NIST's Time

It should be in this format:

``` {.default}
NIST time  : 3874089043
```

### 5. Print Out the System Time

Here's some Python code that get the number of seconds since January 1
1900 00:00 from your system clock.

You can paste this right into your code and call it to get the system
time.

Print the system time right after the NIST time in the following format:

``` {.default}
System time: 3874089043
```

Here's the code:

``` {.py}
def system_seconds_since_1900():
    """
    The time server returns the number of seconds since 1900, but Unix
    systems return the number of seconds since 1970. This function
    computes the number of seconds since 1900 on the system.
    """

    # Number of seconds between 1900-01-01 and 1970-01-01
    seconds_delta = 2208988800

    seconds_since_unix_epoch = int(datetime.datetime.now().strftime("%s"))
    seconds_since_1900_epoch = seconds_since_unix_epoch + seconds_delta

    return seconds_since_1900_epoch
```

Assuming NIST's number isn't zero:

* If this number is within 10 seconds of NIST's number, that's great.

* If it's within 86,400 seconds, that's OK. And I'd like to hear about
  it because it might be a bug in the above code.

* If it's within a million seconds, I really want to hear about it.

* If it's outside of that, it's probably a bug in your code. Did you use
  `"big"` endian? Are you receiving 4 bytes?

<!-- Rubric

55 points

-5 Program uses TCP sockets
-10 Program connects successfully to time.nist.gov port 37
-10 Program receives data from the server
-5 Program close()s the socket after receiving data
-10 Program properly decodes data from server
-5 Program properly prints out results
-10 Results are within 86,400 seconds of each other

 -->
