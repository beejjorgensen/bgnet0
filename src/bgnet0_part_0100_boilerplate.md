# Foreword
<!-- Beej's guide to Network Concepts
# vim: ts=4:sw=4:nosi:et:tw=72
-->

<!-- No hyphenation -->
<!-- [nh[scalbn]] -->

<!-- Index see alsos -->
<!--
[is[String==>see `char *`]]
[is[New line==>see `\n` newline]]
[is[Ternary operator==>see `?:` ternary operator]]
[is[Addition operator==>see `+` addition operator]]
[is[Subtraction operator==>see `-` subtraction operator]]
[is[Multiplication operator==>see `*` multiplication operator]]
[is[Division operator==>see `/` division operator]]
[is[Modulus operator==>see `%` modulus operator]]
[is[Boolean NOT==>see `!` operator]]
[is[Boolean AND==>see `&&` operator]]
[is[Boolean OR==>see `||` operator]]
[is[Bell==>see `\a` operator]]
[is[Tab (is better)==>see `\t` operator]]
[is[Carriage return==>see `\r` operator]]
[is[Hexadecimal==>see `0x` hexadecimal]]
-->

What is this? Well, it's a guide to a bunch of concepts that you might
see in networking. It's not Network Programming in C---see [flbg[_Beej's
Guide to Network Programming_|bgnet]] for that. But it is here to help
make sense of the terminology, and also to do a bit of network
programming in Python.

Is it _Beej's Guide to Network Programming in Python_? Well, kinda,
actually. The C book is more about how C's (well, Unix's) network API
works. And this book is more about the concepts underlying it, using
Python as a vehicle.

I trust that's perfectly confusing. Maybe just skip to the _Audience_
section, below.

## Audience

Are you completely new to networking and are confused by all these terms
like ISO-OSI, TCP/IP, ports, Ethernet, LANs, and all that? And maybe you
want to write some network-capable code in Python? Congrats! You're the
target audience!

But be forewarned: this guide assumes that you've already got some
Python programming knowledge under your belt.

## Official Homepage

This official location of this document is (currently)
[fl[https://beej.us/guide/bgnet0/|https://beej.us/guide/bgnet0/]].

## Email Policy

I'm generally available to help out with email questions so feel free to
write in, but I can't guarantee a response. I lead a pretty busy life
and there are times when I just can't answer a question you have. When
that's the case, I usually just delete the message. It's nothing
personal; I just won't ever have the time to give the detailed answer
you require.

As a rule, the more complex the question, the less likely I am to
respond. If you can narrow down your question before mailing it and be
sure to include any pertinent information (like platform, compiler,
error messages you're getting, and anything else you think might help me
troubleshoot), you're much more likely to get a response.

If you don't get a response, hack on it some more, try to find the
answer, and if it's still elusive, then write me again with the
information you've found and hopefully it will be enough for me to help
out.

Now that I've badgered you about how to write and not write me, I'd just
like to let you know that I _fully_ appreciate all the praise the guide
has received over the years. It's a real morale boost, and it gladdens
me to hear that it is being used for good! `:-)` Thank you!

## Mirroring

You are more than welcome to mirror this site, whether publicly or
privately. If you publicly mirror the site and want me to link to it
from the main page, drop me a line at
[`beej@beej.us`](mailto:beej@beej.us).

## Note for Translators

If you want to translate the guide into another language, write me at
[`beej@beej.us`](beej@beej.us) and I'll link to your translation from
the main page. Feel free to add your name and contact info to the
translation.

Please note the license restrictions in the Copyright and Distribution
section, below.

## Copyright and Distribution

Beej's Guide to Networking Concepts is Copyright © 2023 Brian "Beej Jorgensen" Hall.

With specific exceptions for source code and translations, below, this
work is licensed under the Creative Commons Attribution-Noncommercial-No
Derivative Works 3.0 License. To view a copy of this license, visit
[`https://creativecommons.org/licenses/by-nc-nd/3.0/`](https://creativecommons.org/licenses/by-nc-nd/3.0/)
or send a letter to Creative Commons, 171 Second Street, Suite 300, San
Francisco, California, 94105, USA.

One specific exception to the "No Derivative Works" portion of the
license is as follows: this guide may be freely translated into any
language, provided the translation is accurate, and the guide is
reprinted in its entirety. The same license restrictions apply to the
translation as to the original guide. The translation may also include
the name and contact information for the translator.

The programming source code presented in this document is hereby granted
to the public domain, and is completely free of any license restriction.

Educators are freely encouraged to recommend or supply copies of this
guide to their students.

Contact [`beej@beej.us`](beej@beej.us) for more information.

## Dedication

The hardest things about writing these guides are:

* Learning the material in enough detail to be able to explain it
* Figuring out the best way to explain it clearly, a seemingly-endless
  iterative process
* Putting myself out there as a so-called _authority_, when really
  I'm just a regular human trying to make sense of it all, just like
  everyone else
* Keeping at it when so many other things draw my attention

A lot of people have helped me through this process, and I want to
acknowledge those who have made this book possible.

* Everyone on the Internet who decided to help share their knowledge in
  one form or another. The free sharing of instructive information is
  what makes the Internet the great place that it is.
* Everyone who submitted corrections and pull-requests on everything
  from misleading instructions to typos.

Thank you! ♥
