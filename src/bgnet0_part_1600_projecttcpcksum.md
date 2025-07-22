# Project: Validating a TCP Packet

In this project you'll write some code that validates a TCP packet,
making sure it hasn't been corrupted in transit.

**Inputs**: A sequence of pairs of files:

* One contains the source and destination IPv4 addresses in
  dots-and-numbers notation.

* The other contains the raw TCP packet, both the TCP header and the
  payload.

You can [fls[download the input files from the exercises
folder|tcpcksum/tcp_data.zip]].

**Outputs**:

* For each pair of files, print `PASS` if the TCP checksum is correct.
  Otherwise print `FAIL`.

There are a lot of parts to this project, so it is suggested you write
and test as you go.

**You should understand this specification 100% before you begin to plan
your approach! Get clarification before proceeding to the planning
stage!**

**The ABSOLUTE HARDEST PART of this project is understanding it! Your
code will never work before you understand it!**

**The model solution is 37 lines of code!** (Not including whitespace
and comments.) This is not a number to beat, but is an indication of how
much effort you need to put in to understanding this spec versus typing
in code!

## Banned Functions

You may not use anything in the `socket` library.

## How To Code This

You can do this however you like, but I recommend this order. Details
for each of these steps is included in the following sections.

1. Read in the `tcp_addrs_0.txt` file.
1. Split the line in two, the source and destination addresses.
1. Write a function that converts the dots-and-numbers IP addresses into
   bytestrings.
1. Read in the `tcp_data_0.dat` file.
1. Write a function that generates the IP pseudo header bytes from the
   IP addresses from `tcp_addrs_0.txt` and the TCP length from the
   `tcp_data_0.dat` file.
1. Build a new version of the TCP data that has the checksum set to
   zero.
1. Concatenate the pseudo header and the TCP data with zero checksum.
1. Compute the checksum of that concatenation
1. Extract the checksum from the original data in `tcp_data_0.dat`.
1. Compare the two checksums. If they're identical, it works!
1. Modify your code to run it on all 10 of the data files. **The first
   5 files should have matching checksums! The second five files should
   not!** That is, the second five files are simulating being corrupted
   in transit.

## Checksum in General

The checksum in TCP is a 16-bit value that represents a "sum total" of
all the bytes in the packet. (Plus a bit more. And it's not just the
total of the bytes--don't just add them up!!! More below.)

The TCP header itself contains the checksum from the sending host.

The receiving host (which is what you're pretending to be, here)
computes its own checksum from the incoming data, and makes sure it
matches the value in the packet.

If it does, the packet is good. If it doesn't, it means something got
corrupted. Either the source or destination IP addresses are wrong, or
the TCP header is corrupted, or the data is wrong. Or the checksum
itself got corrupted.

In any case, the TCP software in the OS will request a resend if the
checksums don't match.

Your job in this project is to re-compute the checksum of the given
data, and make sure it matches (or doesn't) the checksum already in the
given data.

## Input File Details

**Download [fls[this ZIP|tcpcksum/tcp_data.zip]] with the input files**.

There are 10 sets of them. The first 5 have valid checksums. The second
5 are corrupted.

In case you didn't notice, the previous line tells you what you have to
do to get 100% on this project!

The files are named like so:

``` {.default}
tcp_addrs_0.txt
tcp_addrs_0.dat

tcp addrs_1.txt
tcp addrs_1.dat
```

and so on, up to the index number 9. Each pair of files is related.

### The `.txt` File

You can look at the `tcp_addrs_n.txt` files in an editor and you'll see
it contains a pair of random IP addresses, similar to the following:

``` {.default}
192.0.2.207 192.0.2.244
```

These are the _source IP address_ and _destination IP address_ for this
TCP packet.

Why do we need to know IP information if this is a TCP checksum? Stay
tuned!

### The `.dat` File

This is a binary file containing the raw TCP header followed by payload
data. It'll look like garbage in an editor. If you have a hexdump
program, you can view the bytes with that. For example, here's the
output from `hexdump`:

``` {.sh}
hexdump -C tcp_data_0.dat
```

``` {.default}
00000000  3f d7 c9 c5 ed d8 23 52  6a 15 32 96 50 d9 78 d8  |?.....#Rj.2.P.x.|
00000010  67 be ba aa 2a 63 25 2d  7c 4f 2a 39 52 69 4b 75  |g...*c%-|O*9RiKu|
00000020  42 39 53                                          |B9S|
00000023
```

But for this project, the only things in that file you really will care
about are:

* The length (in bytes) of the data
* The 16-bit big-endian checksum that's stored at offset 16-17

More on that later!

Note: these files contain "semi-correct" TCP headers. All the parts are
there, but the various values (especially in the flags and options
fields) might make no sense.

## How On Earth Do You Compute A TCP Checksum?

It's not easy.

The TCP checksum is there to verify the integrity of several things:

* The TCP header
* The TCP payload
* The source and destination IP addresses (to protect against misrouted
  data ending up in the TCP stream).

The last part is interesting, because the IP addresses aren't in the TCP
header or data at all, so how do we get them included in the checksum?

A TCP checksum is a two-byte number that is computed like this, given
TCP header data, the payload, and the source and destination IP
addresses:

* Build a sequence of bytes representing the IP Pseudo header (see
  below).

* Set the existing TCP header checksum to zero.

* Concatenate the IP pseudo header + the TCP header and payload.

* Compute the checksum of that concatenation.

That's how you compute a TCP checksum.

But there are a lot of details.

## The IP Pseudo Header

Since we want to make sure that the IP addresses are correct for this
data, as well, we need to include the IP header in our checksum.

Except we don't include the real IP header. We make a fake one. And it
looks like this (stolen straight out of the TCP RFC):

``` {.default}
+--------+--------+--------+--------+
|           Source Address          |
+--------+--------+--------+--------+
|         Destination Address       |
+--------+--------+--------+--------+
|  Zero  |  PTCL  |    TCP Length   |
+--------+--------+--------+--------+
```

Don't let the grid layout fool you: the IP pseudo header is a string of
bytes. It's just in this layout for easier human consumption.

Each `+` sign in diagram represents a byte delimiter.

So the Source Address is 4 bytes. (Hey! IPv4 addresses are 4 bytes
long!) **You get this out of the `tcp_addrs_n.txt` files**.

The Destination Address is 4 bytes. **You get this out of the
`tcp_addrs_n.txt` files, as well**.

Zero is one byte, just set to byte value `0x00`.

PTCL is the protocol, and that is always set to a byte value of `0x06`.
(IP has some magic numbers that represent the higher level protocol
above it. TCP's number is 6. That's where it comes from.)

TCP Length is the total length, in bytes of the TCP packet and data,
big-endian. **This is the length of the data you'll read out of the
`tcp_data_n.dat` files**.

So before you can compute the TCP checksum, you have to fabricate an IP
pseudo header!

### Example Pseudo Header

If the source IP is `255.0.255.1` and the dest IP is `127.255.0.1`, and
the TCP length is 3490 (hex 0x0da2), the pseudo header would be this
sequence of bytes:

``` {.default}
 Source IP |  Dest IP  |Z |P |TCP length
           |           |  |  |  
ff 00 ff 01 7f ff 00 01 00 06 0d a2
```

`Z` is the zero section. And `P` is the protocol (always 6).

See how the bytes line up to the inputs? (255 is hex 0xff, 127 is hex
0x7f, etc.)

### Getting the IP Address Bytes

If you noticed, the IP addresses in the `tcp_addrs_n.txt` files are in
dots and numbers format, not binary.

**You're going to need to write a function that turns a dots-and-numbers
string into a sequence of 4 bytes.**

Algorithm:

* Split the dots and numbers into an array of 4 integers.
* Convert each of those to a byte with `.to_bytes()`
* Concatenate them all together into a single bytestring.

This function will take a dots-and-numbers IPv4 address and return a
four-byte bytestring with the result.

Here's a test:

* Input: `"1.2.3.4"`
* Output: `b'\x01\x02\x03\x04'`

Then you can run this function on each of the IP addresses in the input
file and append them to the pseudoheader.

### Getting the TCP Data Length

This is easy: once you read in one of the `tcp_data_n.dat` files, just
get the length of the data.

``` {.py}
with open("tcp_data_0.dat", "rb") as fp:
    tcp_data = fp.read()
    tcp_length = len(tcp_data)  # <-- right here
```

**Be sure to use `"rb"` when reading binary data! That's what the `b` is
for! If you don't do this, it might break everything!**

## The TCP Header Checksum

When computing the checksum, we need a TCP header with its checksum
field set to zero.

And we also need to extract the existing checksum from the TCP header we
received so that we can compare it against the one we compute!

This diagram is massive, but we actually only care about one part. So
skim this and move on to the next paragraph:

``` {.default}
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

(Again, like the IP pseudo header, this is encoded as a stream of
bytes and the grid is only here to make our human lives easier. This
grid is 32-bits across, numbered along the top.)

See where it says `Checksum`? That's the bit we care about for this
project. And it's a two-byte number (big-endian) at byte offsets 16 and
17 inside the TCP header.

It will also be at byte offset 16-17 in the files `tcp_data_n.dat` since
those files start with the TCP header. (Followed by the TCP payload.)

**You'll need the checksum from that file. Use a slice to get those two
bytes and then use `.from_bytes()` to convert it into a number. This is
the original checksum you'll compare against at the end of the day!**

**You'll also need to generate a version of that TCP header and data
where the checksum is set to zero. You can do it like this:**

``` {.py}
tcp_zero_cksum = tcp_data[:16] + b'\x00\x00' + tcp_data[18:]
```

See how we made a new version of the TCP data there? We sliced
everything before and after the existing checksum, and put two zero
bytes in the middle.

## Actually Computing the Checksum

We're getting there. So far we've done these steps:

* Build the IP pseudo header
* Extract the checksum from the existing TCP header
* Build a version of the TCP header with a zero checksum

And now we get to do the math. Here's what the spec says to do:

> The checksum field is the 16 bit one's complement of the one's
> complement sum of all 16 bit words in the header and text.

All right, we're already in trouble. The what of the what?

One's complement is a way of representing positive and negative integers
in binary. We don't need to know the details for this, gratefully.

But one thing we do need to notice is that we're talking about all the
"16 bit words"... what is that?

It means that instead of considering all this data to be a bunch of
bytes, we're considering it to be a bunch of 16-bit values packed
together.

So if you have the bytes:

``` {.default}
01 02 03 04 05 06
```

we're going to think of that as 3 16-bit values:

``` {.default}
0102 0304 0506
```

And we're going to be adding those together. With one's complement
addition. Whatever that means.

Hey--but what if there are an odd number of bytes?

> If a segment contains an odd number of header and text octets to be
> checksummed, the last octet is padded on the right with zeros to form
> a 16 bit word for checksum purposes.

So we're going to have to look at the `tcp_length` we got from taking
the length of the data from the `tcp_data_0.dat` file. If it's odd, just
add a zero to the end of the entire data.

Conveniently, we have a copy of the TCP data we can use already: the
version we made with the checksum zeroed out. Since we're going to be
iterating over this anyway, might as well append the zero byte to that:

``` {.py}
if len(tcp_zero_cksum) % 2 == 1:
    tcp_zero_cksum += b'\x00'
```

That will force it to be even length.

We can extract all those 16-bit values doing something like the
following. Remember that the data to be checksummed includes the pseudo
header and TCP data (with the checksum field set to zero):

``` {.py}
data = pseudoheader + tcp_zero_cksum

offset = 0   # byte offset into data

while offset < len(data):
    # Slice 2 bytes out and get their value:

    word = int.from_bytes(data[offset:offset + 2], "big")

    offset += 2   # Go to the next 2-byte value
```

Great. That iterates over all the words in the whole chunk of data. But
what does that buy us?

What's the checksum, already?

Let's take that loop above and add the checksum code to it.

Here we're back to that one's-complement stuff. And some 16-bit stuff,
which is tricky in Python because it uses arbitrary-precision integers.

But here's how we want to do it. In the following example, `tcp_data` is
the TCP data padded to an even length with zero for the checksum.

``` {.py}
# Pseudocode

function checksum(pseudo_header, tcp_data)
    data = pseudo_header + tcp_data

    total = 0

    for each 16-bit word of data:  # See code above

        total += word
        total = (total & 0xffff) + (total >> 16)  # carry around

    return (~total) & 0xffff  # one's complement
```

That "carry around" thing is part of the one's complement math. The
`&0xffff` stuff all over the place is forcing Python to give us 16-bit
integers.

Remember what the spec said?

> The checksum field is the 16 bit one's complement of the one's
> complement sum of all 16 bit words in the header and text.

The loop is getting us the "one's complement sum". The `~total` at the
end is getting us the "one's complement" of that.

## Final Comparison

Now that you've computed the checksum for the TCP data and extracted the
existing checksum from the TCP data, it's time to compare the two.

If they are equal, the data is intact and correct. If they are unequal,
the data corrupted.

In the sample data, the first 5 files are intact, and the last 5 are
corrupted.

## Output

The output of your program should show which TCP data passes and which
fails. That is, this should be your output:

``` {.default}
PASS
PASS
PASS
PASS
PASS
FAIL
FAIL
FAIL
FAIL
FAIL
```

## Success

That was no easy thing. Treat yourself! You earned it.

<!-- Rubric

100 points

10
Function converts dots-and-numbers to 4 big-endian bytes

5
Pseudo header contains source and destination IP addresses in binary format

5
Pseudo header contains zero in zero field

5
Psudeo header contains binary value 0x06 in the protocol field

5
Pseudo header contains proper TCP segment length as a 16-bit big-endian number

5
Checksum concatenates the pseudo header and TCP header and data

5
Checksum pads complete data to an even byte length

10
Checksum computation operates on 16-bit words

10
Checksum computation is correct one's complement math

5
Checksum is inverted before returning

5
Checksum results are masked correctly to 16 bits

10
All 10 file sets are read and processed

5
Original checksum is extracted correctly

5
A version of the TCP data with checksum zeroed is constructed correctly

10
"PASS"/"FAIL" output is correct.

-->
