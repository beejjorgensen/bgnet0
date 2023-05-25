<!-- IP, Subnet, and Subnet Mask Review -->

<!--
Documentation subnets: 192.0.2.0/24, 198.51.100.0/24, 203.0.113.0/24
-->

# IP Subnets and Subnet Masks

[_Everything in this chapter will use IPv4, not IPv6. The concepts
are basically the same; it's just easier to learn with IPv4._]

**If you're needing to review your Bitwise Operations, please see
the [Appendix: Bitwise Operations](#appendix-bitwise).**

In this chapter:

* Address representation
* Converting from dots-and-numbers to a value
* Converting from a value to dots-and-numbers
* Subnet and host refresher
* Subnet Mask refresher
* Finding the subnet mask from slash notation
* Finding the subnet for an IP address, given that address and a subnet
  mask

## Address Representation

Remember that IPv4 addresses are commonly shown in dots-and-numbers
notation, e.g. `198.51.100.10`.

And recall that each of those numbers is a byte, which can have values
0-255, decimal (which is the same as 00-FF hexadecimal, and
00000000-11111111 binary).

So really, the dots-and-numbers is just there for our human convenience.

> Protip: remember that different number bases like hex, binary, and
> decimal are just different ways of writing a value down. Kind of like
> different languages for representing the same numeric value.
>
> When a value is stored in a variable, it's best to think of it as
> existing in a pure numeric sense--no base at all. It's only when you
> write it down (in code or print it out) that the base matters.
>
> For instance, Python prints everything in decimal (base 10) by
> default. It has various methods to override that and output in another
> base.

Let's look at that address `198.51.100.10` in hex: `c6.33.64.0a`.

Now let's cram those bytes together into a single hex number:
`c633640a`.

Converting `c633640a` into decimal, we get: `3325256714`.

So in a way, these all represent the same IP address:

``` {.default}
198.51.100.10
c6.33.64.0a
c633640a
3325256714
```

Fair enough?

But why?

Well, we're about to do some math on IP addresses. Now, we _could_ do
that math one byte at a time and it would work just fine.

But it turns out that if we pack all those bytes into a single number,
we can do the math on all the bytes at once, and it becomes easier. Stay
tuned!

## Converting from Dots-and-Numbers

Our goal in this section is to take a dots-and-numbers string like
`198.51.100.10` and convert it into the corresponding value, like
`3325256714` (decimal).

We could do it with string manipulation like in the previous section,
but let's do it in a more _bitwise_ sense.

Let's extract the individual numbers from an IP address (Python can do
this with `.split()`.)

The string:

``` {.py}
"198.51.100.10"
```

becomes the list of strings:

``` {.py}
["198", "51", "100", "10"]
```

Now let's convert each of those to integers. (Python could do this with
a loop and the `int()` function, or `map()`, or a list comprehension.)

``` {.py}
[198, 51, 100, 10]
```

Now I'm going to write these numbers in hex because it makes the future
steps more clear. But remember that they're just stored as numeric
values, so Python won't print them as hex unless you ask it to.

``` {.py}
[0xc6, 0x33, 0x64, 0x0a]
```

To build our number, we're going to rely on a couple bitwise operations:
bitwise-OR and bitwise-shift.

For the sake of example, let's hardcode the math:

``` {.py}
(0xc6 << 24) | (0x33 << 16) | (0x64 << 8) | 0x0a
```

Running that in Python gives the decimal number `3325256714`. Converted
to hex, we're back to `0xc633640a`.

You can use the above formula to convert any set of 4 bytes to a packed
number.

> There's also a clever loop you can run to do it one byte at a time.
> See if you can figure that out as an added challenge! DM the
> instructor to see how clever you were.

## Converting to Dots-and-Numbers

What if you have that value from the previous section, `3325256714`, and
you want to get it back as an IP address?

We can use some shifting to make that happen, too! But we have to do
some AND masking to get just the parts of the number we want.

Let's convert to hex because each byte is exactly 2 hex digits and it's
a little easier to see them: `0xc633640a`.

Now let's look at that number shifted by 0 bits, 8 bits, 16 bits, and 24
bits:

``` {.py}
0xc633640a >> 24 == 0x000000c6
0xc633640a >> 16 == 0x0000c633
0xc633640a >> 8  == 0x00c63364
0xc633640a >> 0  == 0xc633640a
```

If you look at just the two digits on the right, you'll see they're the
bytes of the original number:


``` {.py}
0xc633640a >> 24 == 0x000000 c6
0xc633640a >> 16 == 0x0000c6 33
0xc633640a >> 8  == 0x00c633 64
0xc633640a >> 0  == 0xc63364 0a
```

So we're onto something, except looking at the right shift 8, for
example, we get this:

``` {.py}
0xc633640a >> 8  == 0x00c63364
```

So yes, I'm interested in the byte `0x64` like we see on the right, but
not the `0xc633` part of it. How can I zero that higher part out,
leaving just the `0x64`?

We can use an _AND mask_! The bitwise-AND operator can work like a
stencil letting some of the number through and zeroing other parts of
it. Let's do a bitwise-AND on that number with the byte `0xff`, which is
all 8 bits set to `1` and all bits over the first 8 have implied value
`0`.

``` {.default}
  0x00c63364
& 0x000000ff
------------
  0x00000064
```

Hey! `0x64` is the byte from the IP address we wanted! See how where
there were binary `1`s in the mask (except here represented in hex) it
let the value "show through", while everywhere that had a zero it was
masked out?

Now we can extract our digits:

``` {.py}
(0xc633640a >> 24) & 0xff == 0x000000c6 == 0xc6
(0xc633640a >> 16) & 0xff == 0x00000033 == 0x33
(0xc633640a >> 8)  & 0xff == 0x00000064 == 0x64
(0xc633640a >> 0)  & 0xff == 0x0000000a == 0x0a
```

And those are the individual bytes of the IP address.

To get from there to dots-and-numbers, you can use an f-string or
`.join()` in Python.

## Subnet and Host Refresher

Recall that IP addresses are split into two portions: the subnet number
and the host number. Some of the bits on the left side of the IP address
are the network number, and the remaining bits on the right are the host
number.

Let's look at an example where the left 24 bits (3 bytes) are the
subnet number and the right 8 bits (1 byte) holds the host number.

``` {.default}
 Subnet   | Host
          | 
198.51.100.10
```

So that represents host `10` on subnet `198.51.100.0`. (We replace the
host bits with `0` when talking about the subnet number.)

But I just said above, unilaterally, that there were 24 network bits in
that IP address. That's not very concise. So they invented slash
notation.

``` {.default}
198.51.100.0/24    24 bits are used for subnet
```

Or you could use it with an IP address:

``` {.default}
198.51.100.10/24   Host 10 on subnet 198.51.100.0
```

Let's try it with 16 bits for the network:

``` {.default}
10.121.2.17/16    Host 2.17 on subnet 10.121.0.0
```

Get it?

In those examples we used a multiple of 8 so it would align visually on
a byte boundary, but there's no reason you can't have a fractional part
of a byte left over for a subnet:

``` {.default}
10.121.2.68/28    Host 4 on subnet 10.121.2.64
```

If you don't see where the 4 and 64 came from in the previous example,
try writing the bytes out in binary!

## Subnet Mask Refresher

What is the _subnet mask_? This is a run of `1` bits in binary that
indicates which part of the IP address is the network portion. It is
followed by a run of `0`s in binary that indicate which part is the
host portion.

It's used to determine what subnet an IP address belongs to, or what
part of the IP represents the host number.

Let's draw one in binary. We'll use this example IP address:

``` {.default}
198.51.100.10/24   Host 10 on subnet 198.51.100.0
```

Let's first convert to binary. (There's a hint here that the subnet mask
is a bitwise-AND mask!)

``` {.default}
11000110.00110011.01100100.00001010   198.51.100.10
```

And now, above it, let's draw a run of 24 `1`s (because this is a `/24`
subnet) followed by 8 `0`s (because the IP address is 32 bits total).


``` {.default}
11111111.11111111.11111111.00000000   255.255.255.0  subnet mask!
   
11000110.00110011.01100100.00001010   198.51.100.10
```

That is the subnet mask that corresponds to a `/24` subnet!
`255.255.255.0`!

## Computing the Subnet Mask from Slash Notation

If I tell you a subnet is `/24`, how can you determine the subnet mask
is `255.255.255.0`? Or if I tell you it's `/28` that the mask is
`255.255.255.240`?

For a subnet `/24`, we need a run of 24 `1`s, followed by 8 `0`s.

Look at the [Appendix: Bitwise](#appendix-bitwise) for ways to generate
runs of `1`s in binary and shift them over.

Once you have that big binary number, it's a matter of switching it back
to dots-and-numbers notation, using the technique we outlined, above.

Remember that subnets can end on any bit boundary. `/17` is a fine
subnet. It doesn't have to be a multiple of 8!

## Finding the Subnet for an IP address

If you're given an IP address with slash notation like this:

``` {.default}
198.51.100.10/24   Host 10 on subnet 198.51.100.0
```

How can you extract just the subnet (198.51.100.0) and just the host

You can do it with bitwise-AND!

We can compute the subnet mask for `/24` and get `255.255.255.0`, as
above.

After that, let's take a look in binary and AND these together:

``` {.default}
  11111111.11111111.11111111.00000000   255.255.255.0  subnet mask
& 11000110.00110011.01100100.00001010   198.51.100.10  IP address
-------------------------------------
  11000110.00110011.01100100.00000000   198.51.100.0  network number!
```

We can operate on the whole thing at once instead of a byte at a time,
as well... we just need to cram those numbers together into a single
value, like we did in the section above:

``` {.default}
  11111111111111111111111100000000   255.255.255.0  subnet mask
& 11000110001100110110010000001010   198.51.100.10  IP address
-------------------------------------
  11000110001100110110010000000000   198.51.100.0  network number
```

The AND works on the whole thing at once! Then we can convert back to
dots-and-numbers notation like we did in the previous sections.

Now what if you had the IP address and the subnet mask and wanted to get
the _host_ bits out of the IP address, not the network bits. Do you see
how you could do it? (Hint: bitwise NOT!)

Routers use this all the time--they are given an IP address and they
need to know if it matches any subnets the router is connected to. So it
masks out the IP address's network number and compares it to all the
subnets that router knows. And then forwards it toward the right one.

## Reflect

* What is the 32-bit (4-byte) representation of the IP address
  `10.100.30.90` in hex? In decimal? In binary?
  <!-- 0x0a641e5a, 174333530, 0b1010011001000001111001011010 -->
 
* What is the dots-and-numbers IP address represented by the 32-bit
  number `0xc0a88225`?
  <!-- 192.168.130.37 -->

* What is the dots-and-numbers IP address represented by the 32-bit
  decimal number `180229186`?
  <!-- 10.190.20.66 -->

* What bitwise operations do you need to extract the second byte from
  the left (`0xff`) of the number `0x12ff5678`?
  <!--
  shift, AND
  (n >> 16) & 0xff  or  (n & 0xff0000) >> 16
  -->

* What is the slash notation for subnet mask `255.255.0.0`?
  <!-- /16 -->

* What is the subnet mask for network `192.168.1.12/24`?
  <!-- 255.255.255.0 -->

* What are the numeric operations necessary to convert a slash subnet
  mask to a binary value?
  <!--
  shift, subtract 1
  ((1 << m) - 1) << (32 - m)
  -->

* Given an IP address value (in a single 32-bit number) and a subnet
  mask value (in a single 32-bit number), what bitwise operation do you
  need to perform to get the subnet number from the IP address?
  <!-- AND the two together -->


192.168.1.0 is the network number, but it's not the subnet mask. The subnet mask is the dots-and-numbers value obtained from the slash notation. In this case it's `/24`, which gives us `255.255.255.0`.


