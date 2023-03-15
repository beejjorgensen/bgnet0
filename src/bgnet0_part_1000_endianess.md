# Endianess and Integers

We've done some work transmitting text over the network. But now we want
to do something else: we want to transfer binary integer data.

Sure we _could_ just convert the numbers to strings, but this is more
wasteful than it needs to be. Binary representation is more compact and
saves bandwidth.

But the network can only send and receive bytes! How can we convert
arbitrary numbers to single bytes?

That's what this chapter is all about.

We want to:

* Convert integers to byte sequences
* Convert byte sequences back into integers

And in this chapter we'll look at:

* How numbers are represented by sequences of bytes
* What order those bytes go in
* How to convert a number to a sequence of bytes in Python
* How to convert a sequence of bytes to a number in Python

Key points to look out for:

* Integers can be represented by sequences of bytes.
* We'll convert integers to sequences of bytes before we transmit them
  over the network.
* We'll convert sequences of bytes back into integers when we receive
  them over the network.
* _Big-Endian_ and _Little-Endian_ are two different ways of ordering
  those sequences of bytes.
* Python offers built-in functionality for converting integers to
  sequences of bytes and back again.

## Integer Representations

In this section we'll dive deep into how an integer can be represented
by a sequence of individual bytes.

### Decimal Byte Representation

Let's look at how integers are represented as sequences of bytes. These
sequences of bytes are what we'll send across the network to send
integer values to other systems.

A single byte (in this context well define a byte to be the usual 8
bits) can encode binary values from `00000000` to `11111111`. In
decimal, these numbers go from 0 to 255.

So what happens if you want to store number larger than 255? Like 256?
In that case, you need to use a second byte to store the additional
value.

The more bytes you use to represent an integer, the larger the range of
integers you can represent. One byte can store from 0 to 255. Two bytes
can store from 0 to 65535.

Thinking about it another way, 65536 is the number of combinations of 1s
and 0s you can have in a 16-bit number.

> This section is talking about **non-negative integers** only. Floating
> point numbers use a [different
> encoding](https://en.wikipedia.org/wiki/IEEE_754#Basic_and_interchange_formats).
> Negative integers use a similar technique to positive, but we'll keep
> it simple for now and ignore them.

Let's take a look what happens when we count up from 253 to 259 in a
16-bit number. Since 259 is bigger than a single byte can hold, we'll
use two bytes (holding numbers from 0 to 255), with the corresponding
decimal value represented on the right:

``` {.default}
  0 253   represents 253
  0 254   represents 254
  0 255   represents 255
  1   0   represents 256
  1   1   represents 257
  1   2   represents 258
  1   3   represents 259
```

Notice that the byte on the right "rolled over" from `255` to `0` like
an odometer. It's almost like that byte is the "ones place" and the byte
on the left is the "256s place"... like looking at a base-256 numbering
system, almost.

We could compute the decimal value of the number by taking the first
byte and multiplying it by 256, then adding on the value of the second
byte:

``` {.default}
1  * 256 +   3 = 259
```

Or in this example, where two bytes with values `17` and `178` represent
the value `1920`:

``` {.default}
17 * 256 + 178 = 1920
```

Neither `17` not `178` are larger than `255`, so they both fit in a
single byte each.

So every integer can be perfectly represented by a sequence of bytes.
You just need more bytes in the sequence to represent larger numbers.

### Binary Byte Representations

Binary, hexadecimal, and decimal are just all different "languages" for
writing down values.

So we could rewrite the entire previous section of the document by
merely translating all the decimal numbers to binary, and it would still
be just as true.

In fact, let's do it for the example from the previous section.
Remember: this is numerically equivalent--we just changed the numbers
from decimal to binary. All other concepts are identical.

``` {.default}
00000000 11111101    represents  11111101 (253 decimal)
00000000 11111110    represents  11111110 (254 decimal)
00000000 11111111    represents  11111101 (255 decimal)
00000001 00000000    represents 100000000 (256 decimal)
00000001 00000001    represents 100000001 (257 decimal)
00000001 00000010    represents 100000010 (258 decimal)
00000001 00000011    represents 100000011 (259 decimal)
```

But wait a second--see the pattern? If you just stick the two bytes
together you end up with the exact same number as the binary
representation! (Ignoring leading zeros.)

Really all we've done is take the binary representation of a number and
split it up into chunks of 8 bits. We could take any arbitrary number
like 1,256,616,290,962 decimal and convert it to binary:

``` {.default}
10010010010010100001010101110101010010010
```

and do the same thing, split it up into chunks of 8 bits:

``` {.default}
1 00100100 10010100 00101010 11101010 10010010
```

Since we're packing it into bytes, we should pad that leading `1` out to
8 bits like so:

``` {.default}
00000001 00100100 10010100 00101010 11101010 10010010
```

And there you have it, the byte-by-byte representation of the number
1,256,616,290,962.

### Hexadecimal Byte Representations

Again, it doesn't matter what number base we use--they're just all
different "languages" for representing a numeric value.

Programmers like hex because it's very compatible with bytes (each byte
is 2 hex digits). Let's do the same chart again, this time in hex:

``` {.default}
00 fd    represents 00fd (253 decimal)
00 fe    represents 00fe (254 decimal)
00 ff    represents 00ff (255 decimal)
01 00    represents 0100 (256 decimal)
01 01    represents 0101 (257 decimal)
01 02    represents 0102 (258 decimal)
01 03    represents 0103 (259 decimal)
```

Look at that again! The hex representation of the number is the same as
the two bytes just crammed together! Super-duper convenient.

## Endianess

Ready to get a wrench thrown in the works?

I just finished telling you that a number like (in hex):

``` {.default}
45f2
```

can be represented by these two bytes:

``` {.default}
45 f2
```

But guess what! Some systems will represent `0x45f2` as:

``` {.default}
f2 45
```

It's backwards! This is analogous to me saying "I want 123 pieces of
toast" when in fact I really wanted 321!

There's a name for putting the bytes backward like this. We say such
representations are _little endian_.

This mean the "little end" of the number (the "ones" byte, if I can call
it that) comes at the front end.

The more-normal, more-forward way to write it (like we did at first,
where the number `0x45f2` was reasonably represented in the order `45
f2`) is called _big endian_. The byte in the largest value slot (also
called the _most-significant byte_) is at the front end.

The bad news is that virtually all Intel CPU models are little-endian.

The good news is that Mac M1s are big-endian.

The even better news is that **all network numbers are transmitted as
big-endian**, the sensible way.

Big-endian byte order is called _network byte order_ in network contexts
for this reason.

## Python and Endianess

What if you have some number in Python, how do you convert it into a
byte sequence?

Luckily, there's a built-in function to help with that: `.to_bytes()`.

And there's one to go the other way: `.from_bytes()`

It even allows you to specify the endianess! Since we'll be using this
to transmit bytes over the network, we'll always use `"big"` endian.

### Converting a Number to Bytes

Here's a demo where we take the number 3490 and store it as bytestring
of 2 bytes in big-endian order.

Note that we pass two things into the `.to_bytes()` method: the number
of bytes for the result, and `"big"` if it's to be big-endian, or
`"little"` if it's to be little endian.

``` {.default}
n = 3490

bytes = n.to_bytes(2, "big")
```

If we print them out we'll see the byte values:

``` {.default}
for b in bytes:
    print(b)
```

``` {.default}
13
162
```

Those are the big-endian byte values that make up the number 3490. We
can verify that `13 * 256 + 162 == 3490` easily enough.

If you try to store the number 70,000 in two bytes, you'll get an
`OverflowError`. Two bytes isn't large enough to store values over
65535--you'll need to add another byte.

Let's do one more example in hex:

``` {.py}
n = 0xABCD
bytes = n.to_bytes(2, "big")

for b in bytes:
    print(f"{b:02X}")  # Print in hex
```

prints:

``` {.default}
AB
CD
```

It's the same digits as the original value stored in `n`!

### Convert Bytes Back to a Number

Let's take the full tour. We're going to make a hex number and convert
it to bytes, like we did in the previous section. Then we'll even print
out the bytestring to see what it looks like.

Then we'll convert that bytestring back to a number and print it out to
make sure it matches the original.

``` {.py}
n = 0x0102
bytes = n.to_bytes(2, "big")

print(bytes)
```

gives the output:

``` {.py}
b'\x01\x02'
```

The `b` at the front means this is a bytestring (as opposed to a regular
string) and the `\x` is an escape sequence that appears before a 2-digit
hex number.

Since our original number was `0x0102`, it makes sense that the two
bytes in the byte string have values `\x01` and `\x02`.

Now let's convert that string back and print in hex:

``` {.py}
v = int.from_bytes(bytes, "big")

print(f"{v:04x}")
```

And that prints:

``` {.default}
0102
```

just like our original value!

## Reflect

* Using only the `.to_bytes()` and `.from_bytes()` methods, how can you
  swap the byte order in a 2-byte number? (That is reverse the bytes.)
  How can you do that without using any loops or other methods? (Hint:
  `"big"` and `"little"`!)

* Describe in your own words the difference between Big-Endian and
  Little-Endian.

* What is Network Byte Order?

* Why not just send an entire number at once instead of breaking it into
  bytes?

* Little-endian just seems backwards. Why does it even exist?  Do a
  little Internet searching to answer this question. 

