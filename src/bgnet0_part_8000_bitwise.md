# Appendix: Bitwise Operations {#appendix-bitwise}

In this section, we'll refresh on _bitwise operations_.

The bitwise operators in a language manipulate the bits of numbers.
These operators act as if a number is represented in binary, even if its
not. They can work on numbers of any base in Python code, but it makes
the most sense to us as humans in binary.

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

We'll look at:

* Coding different bases in Python
* Printing different bases in Python
* Bitwise-AND
* Bitwise-OR
* Bitwise-NOT
* Bitwise shift
* Setting a given number of `1` bits

## Coding Different Bases in Python

What value is `110101`?

Your brain might think I'm asking "What is this binary number in
decimal?" But you'd be wrong!

I did not specify a number base along with that number, so you don't
know if it's binary or decimal or hex!

For the record, I meant it to be decimal, so it's one hundred ten
thousand one hundred one.

Python and most other languages assume the numbers in your code are
decimal unless you specify otherwise.

If you wanted it to be a different base, you have to prefix the number
to indicate the base.

These prefixes are:

<!-- CAPTION: Python Number Base Prefixes -->
|Base|Base name|Prefix|
|-|-|-|
|2|Binary|`0b`|
|8|Octal|`0o`|
|10|Decimal|None|
|16|Hexadecimal|`0x`|

So let's write some numbers in different bases:

``` {.default}
  110101 decimal!
0b110101 binary!
0x110101 hex!
```

Let's look at the decimal value `3490`. I can convert that to hex and
get `0xda2`.

It's important to remember these two values are identical:

``` {.py}
>>> 3490 == 0xda2
True
```

It's just a different "language" for representing the same value.

## Printing and Converting

Remember that there's no such thing as a value "stored in hex" or
"stored in decimal" in a variable. The variable holds just a numeric
value and we shouldn't consider it to have a base.

It only acquires a base when we write in our code or print it out. Then
we have to specify the base. (Although Python uses decimal by default in
all cases.)

You can convert a value to a hex string with the `hex()` function. All
"converted" values end up as strings. What else would they be?

``` {.py}
>>> print(hex(3490))
0xda2
```

You can convert a value to a binary string with the `bin()` function.

``` {.py}
>>> print(bin(3490))
0b110110100010
```

You can also use f-strings to get the job done:

``` {.py}
>>> print(f"3490 is {3490:x} in hex and {3490:b} in binary")
3490 is da2 in hex and 110110100010 in binary
```

The f-strings have a nice feature of being able to pad to a field width
with zeros. Let's say you wanted an 8-digit hex number representation,
you could do it like this:

``` {.py}
>>> print(f"3490 is {3490:08x} in hex")
3490 is 00000da2 in hex
```

## Bitwise-AND

Bitwise-AND mashes two numbers together with an AND operation. The result of
an AND operation is `1` if both of the two input bits are `1`.
Otherwise it's `0`. 

<!-- CAPTION: Bitwise-AND Truth Table -->
|A|B|A AND B|
|:-:|:-:|:-:|
|0|0|0|
|0|1|0|
|1|0|0|
|1|1|1|

Let's do an example and AND two binary numbers together. Bitwise-AND uses
the ampersand operator (`&`) in Python and many other languages.

``` {.default}
  0     0     1     1
& 0   & 1   & 0   & 1
---   ---   ---   ---
  0     0     0     1
```

You see the result is `1` only if both of the two input bits are `1`.

Larger inputs are AND'd by pairs of individual bits, which are what
appears in each column of the large numbers below. (For completeness, the
decimal result is shown on the right, but this is derived from the binary
representation. It's not easy to look at two decimal numbers and
ascertain their bitwise-AND.)

``` {.default}
  0100011111000101010      146986
& 1001111001001111000    & 324216
---------------------    --------
  0000011001000101000       12840
```

See how any particular output bit is `1` only if both bits in the column
above it are `1`.

## Bitwise-OR

Bitwise-OR mashes two numbers together with an OR operation. The result of
an OR operation is `1` if either or both of the two input bits are `1`.
Otherwise it's `0`. 

<!-- CAPTION: Bitwise-OR Truth Table -->
|A|B|A OR B|
|:-:|:-:|:-:|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|1|

Let's do an example and OR two binary numbers together. Bitwise-OR uses
the pipe operator (`|`) in Python and many other languages.

``` {.default}
  0     0     1     1
| 0   | 1   | 0   | 1
---   ---   ---   ---
  0     1     1     1
```

You see the result is `1` if either of the two input bits are `1`.

Larger inputs are OR'd by pairs of individual bits, which are what
appears in each column of the large numbers below. (For completeness, the
decimal result is shown on the right, but this is derived from the binary
representation. It's not easy to look at two decimal numbers and
ascertain their bitwise-OR.)

``` {.default}
  0100011111000101010      146986
| 1001111001001111000    | 324216
---------------------    --------
  1101111111001111010      458362
```

See how any particular output bit is `1` if either or both bits in the
column above it are `1`.

## Bitwise-NOT

Bitwise-NOT _inverts_ a value by changing all the `1` bits to `0` and
all the `0` bits to `1` This is a unary operator--it just works on a
single number.

<!-- CAPTION: Bitwise-NOT Truth Table -->
|A|NOT A|
|:-:|:-:|
|0|1|
|1|0|

Let's do an example and NOT a single bit number. Bitwise-NOT uses
the tilde operator (`~`) in Python and many other languages.

``` {.default}
~ 0   ~ 1
---   ---
  1     0
```

You see the result is simply flipped to the other bit value.

Larger inputs are NOT'd by individual bits, which are what appears in
each column of the large numbers below. (For completeness, the decimal
result is shown on the right, but this is derived from the binary
representation. It's not easy to look at two decimal numbers and
ascertain their bitwise-NOT.)

``` {.default}
~ 0100011111000101010    ~ 146986
---------------------    --------
  1011100000111010101      377301
```

See how any particular output bit is `1` only if both bits in the column
above it are `1`.

Python note: the bitwise-NOT of a number will frequently be negative
because of Python's arbitrary-precision arithmetic. If you need it to be
positive, bitwise-AND the result with the number of `1` bits you need to
represent the final value. For instance, to get a byte with `37`
inverted, you can do any of these:

``` {.py}
(~37) & 255
(~37) & 0xff
(~37) & 0b11111111
```

(Because, of course, those are all the same number in different bases!)

## Bitwise shift

This is an interesting one. Using this, you can move a number back and
forth by a certain number of bits.

Check out how we're moving all the bits left by 2 in this example:

``` {.default}
000111000111  left shifted by 2 is:
011100011100
```

Or we can shift right:

``` {.default}
000111000111  right shifted by 2 is:
000001110001  
```

New bits on the left or right are set to zero, and bits that fall off
the ends vanish forever.

> I'm lying a little because of how a lot of languages handle right
> shifting of negative numbers. If you shift a negative number right,
> the new bits might be `1` instead of `0` depending on the language.
> Python uses arbitrary precision integer math, so there are actually an
> infinite number of `1` bits on the left of any negative number in
> Python, making things even weirder.
>
> For now, best to just think about positive numbers.

The operator is `<<` for left shift and `>>` for right shift in most
languages (sorry, Ruby!). Let's do an example:

``` {.py}
>>> v = 0b00000101
>>> print(f"{v << 2:08b}")
00010100
```

There we had a byte in binary and we left-shifted it by 2 with `v << 2`.
And you can see the `101` has moved over 2 bits to the left!

## Setting a Given Number of `1` bits

This is a little bit-hack we can do if we want to get a number with a
certain number of contiguous bits set to `1`.

For example, what if I want a number with 12 bits set to one, namely:

``` {.py}
0b111111111111
```

What would I have to do?

We can make use of a couple tricks here. Let's start with trick #1.

If you want to set the nth bit to `1` (where the rightmost bit is bit
number 0), you can raise 2 to that power and get there.

Let's set bit #5 to 1. We can take `2**5` which gives us `32`. And `32`
in binary is `100000`. There we are.

Another option is to left-shift a `1` by 5 bits: `1 << 5` is `100000` in
binary, which is `32` decimal.

That works.

But how do we get to our bit run of `1`s from there?

Check this out: what's 32 minus 1? 31. Not a trick question. But let's
look at those in binary:

``` {.default}
32   100000
31   011111
```

Hey! It's a run of `1`s! Not only that, but it's a run of 5 `1`s, just
like we wanted! (This is analogous to subtraction in decimal. 10,000 - 1
is 9,999. Just in binary we roll over to all `1`s, not `9`s.)

``` {.py}
run_of_ones = (1 << count) - 1
```

Here's our run of 12 `1`s:

``` {.py}
>>> bin((1 << 12) - 1)
'0b111111111111'
```

If you like these sorts of _bit-twiddling hacks_, you might enjoy the
book [Hacker's
Delight](https://en.wikipedia.org/wiki/Hacker%27s_Delight). Chapter 2,
which covers a lot of these techniques had historically been distributed
for free; you might be able to find a PDF floating around.

## Reflect

* What is `2342 & 2332`?

* What is `0b110101 | 112`?

* What is `~0b101010010101` in binary? (Python will show the result as a
  negative number, but you can turn it back positive by ANDing it with
  `0b111111111111`. And don't forget that Python leaves off leading
  zeroes when it prints!)

* What is `16 << 1`?

* What is `64 << 1`?

* What is `4200 << 1`? See the pattern?

* What is `16 >> 2`?

* What is `0b11100111 << 3`?

* What is `(1 << 8) - 1`?

* What is `0x01020304 & ((1 << 16) - 1)`?

<!--

Answers:

2342 & 2332
2308	0x904	0b100100000100

0b110101 | 112
117	0x75	0b1110101

(~0b101010010101)&0b111111111111
1386	0x56a	0b10101101010

16 << 1
32

64 << 1
128

4200 << 1
8400   Doubles

16 >> 2
4

0b11100111 << 3
1848	0x738	0b11100111000

 ~ %  (1 << 8) - 1
255	0xff	0b11111111

0x01020304 & ((1 << 16) - 1)
772	0x304	0b1100000100

-->
