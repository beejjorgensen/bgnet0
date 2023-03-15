# Project: The Word Server

This is a project that is all about reading packets.

You'll receive a stream of encoded data from a server (provided) and
you'll have to write code to determine when you've received a complete
packet and the print out that data.

## Overview

**First:** download these files:

* [fls[**`wordserver.py`**|word/wordserver.py]]: a ready-to-run
  server that hands out lists of random words.

* [fls[**`wordclient.py`**|word/wordclient.py]]: skeleton code for the client.

**RESTRICTION! Do not modify any of the existing code!** Just search for
`TODO` and fill in that code. You may add additional functions and
variables if you wish.

**REQUIREMENT! The code should work with any positive value passed to
`recv()` between `1` and `4096`!** You might want to test values like
`1`, `5`, and `4096` to make sure they all work.

**REQUIREMENT! The code must work with words from length 1 to length
65535.** The server won't send very long words, but you can modify it to
test. To build a string in Python of a specific number of characters,
you can:

``` {.py}
long_str = "a" * 256   # Make a string of 256 "a"s
```

**PROTIP! Read and understand all the existing client code before you
start.** This will save you all kinds of trouble. And note how the
structure of the main code doesn't even care about bytes and
streams--it's only concerned with entire packets. Cleaner, right?

You are going to complete two functions:

* **`get_next_word_packet()`**: gets the next complete word packet from
  the stream. This should return the _complete packet_, the header and
  the data.

* **`extract_word()`**: extract and return the word from a complete word
  packet.

What do they do? Keep reading!

## What is a "Word Packet"?

When you connect to the server, it will send you a stream of data.

This stream is made up of a random number (1 to 10 inclusive) of words
prefixed by the length of the word in bytes.

Each word is UTF-8 encoded.

The length of the word is encoded as a big-endian 2-byte number.

For example, the word "hello" with length 5 would be encoded as the
following bytes (in hex):

``` {.default}
length 5
  |
+-+-+
|   | h  e  l  l  o
00 05 68 65 6C 6C 6F
```

The numbers corresponding to the letters are the UTF-8 encoding of those
letters.

> Fun fact: for alphabetic letters and numbers, UTF-8, ASCII, and
> ISO-8859-1 are all the same encoding.

The word "hi" followed by "encyclopedia" would be encoded as two word
packets, transmitted in the stream like so:

``` {.default}
length 2   length 12
  |           |
+-+-+       +-+-+
|   | h  i  |   | e  n  c  y  c  l  o  p  e  d  i  a
00 02 68 69 00 0C 65 6E 63 79 63 6C 6F 70 65 64 69 61
```

## Implementation: `get_next_word_packet()`

This function takes the connected socket as argument. It will return the
next word packet (the length plus the word, as received) as a
bytestring.

It will follow the process outlined in the [Parsing Packets
chapter](#parsing-packets) for extracting packets from a stream of data.

For example, if the words were "hi" and "encyclopedia" from the example
above, and we received the first 5 bytes, the packet buffer would
contain:

``` {.default}
      h  i  
00 02 68 69 00
```

We see the first word is 2 bytes long, and we have captured those 2
bytes.

We would extract and return the first word ("hi") and its length (bytes
`00` `02`) and return this bytestring:

``` {.default}
      h  i
00 02 68 69
```

We'd also strip those bytes out of the packet buffer so that all that
remained was the zero that was at the end.

``` {.default}
00
```

At that point, lacking a complete word in the buffer, a subsequent call
to the function would trigger a `recv(5)` for the next chunk of data,
giving us:

``` {.default}
      e  n  c  y
00 0C 65 6E 63 79
```

And so on.

## Implementation: `extract_word()`

This function takes a complete word packet as input, such as:

``` {.default}
      h  i
00 02 68 69
```

and returns a string of the word, `"hi"`.

This involves slicing off everything past the two length bytes to get
the word bytes.

But remember: the word is UTF-8 encoded! So you have to call `.decode()`
to turn it back into a string. (The default decoding is UTF-8, so you
don't have to pass an argument to `.decode()`.)

<!-- Rubric

55 points

-10 Code that was required to be unmodified was not modified
-5 Recv only called with 5 as the argument
-15 Function get_next_word_packet() returns only the next single complete packet as a bytestring
-5 Function get_next_word_packet() properly removes the complete single packet from the front of the global buffer
-5 Function get_next_word_packet() returns None when the connection was closed by the server
-5 Function extract_word() extracts the word from the packet
-10 Function extract_word() returns the word as a decoded UTF-8 string

 -->
