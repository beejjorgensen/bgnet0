# Project: Digging DNS Info

The `dig` utility is a great command line tool for getting DNS
information from your default name server, or from any name server.

## Installation

On Macs:

```
brew install bind
```

On WSL:

```
sudo apt install dnsutils
```

## Try it Out

Type:

```
dig example.com
```

and see what it gives back.

```
; <<>> DiG 9.10.6 <<>> example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60465
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;example.com.                   IN      A

;; ANSWER SECTION:
example.com.            79753   IN      A       93.184.216.34

;; Query time: 23 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Fri Nov 18 18:37:13 PST 2022
;; MSG SIZE  rcvd: 56
```

That's a lot of stuff. But let's look at these lines:

```
;; ANSWER SECTION:
example.com.            79753   IN      A       93.184.216.34
```

That's the IP address for `example.com`! Notice the `A`? That means this
is an address record.

You can get other record types, as well. What if you want the mail
exchange server for `oregonstate.edu`? You can put that on the command line:

```
dig mx oregonstate.edu
```

We get:

```
;; ANSWER SECTION:
oregonstate.edu. 600 IN MX 5 oregonstate-edu.mail.protection.outlook.com.
```

Or if we want the name servers for `example.com`, we can:

```
dig ns example.com
```

## Time To Live (TTL)

If you `dig` an `A` record, you'll see a number on the line:

```
;; ANSWER SECTION:
example.com.            78236   IN      A       93.184.216.34
```

In this case, it's `78236`. This is the TTL of an entry in the cache.
This is telling you that the name server you used has cached that IP
address, and it won't expire that cache entry until 78,236 more seconds
have elapsed. (For reference, there are 86,400 seconds in a day.)

## Authoritative Servers

If you get an entry that's cached in your name server, you'll see
`AUTHORITY: 0` in the `dig` output:

```
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
                                            ^^^^^^^^^^^^
```

But if the entry comes directly from the name server that's responsible
for the domain, you'll see `AUTHORITY: 1` (or some positive number).

## Getting the Root Name Servers

Just type `dig` and hit return and you'll see all the root DNS servers,
probably both their IPv4 (record type `A`) and IPv6 (record type `AAAA`)
addresses

## Digging at a Specific Name Server

If you know a name server that you want to query, you can use an `@`
sign to specify that server.

Two popular free-to-use name servers are `1.1.1.1` and `8.8.8.8`. 

Let's ask one of them for the IP of `example.com`

```
dig @8.8.8.8 example.com
```

And we get our expected answer. (Though the TTL is probably
different--these are different servers and their cache entries have
different ages, after all.)

## Digging at a Root Name Server

There were a bunch of root name servers, so let's dig `example.com` at
one of them:

```
dig @l.root-servers.net example.com
```

We get some interesting results:

```
com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
com.                    172800  IN      NS      c.gtld-servers.net.
com.                    172800  IN      NS      d.gtld-servers.net.
```

and then some.

That's not `example.com`... and look--those are `NS` records, name
servers.

This is the root server telling us, "I don't know who `example.com` is,
but here are some name servers that know what `.com` is."

So we choose one of those and `dig` there:

```
dig @c.gtld-servers.net example.com
```

And we get:

```
example.com.            172800  IN      NS      a.iana-servers.net.
example.com.            172800  IN      NS      b.iana-servers.net.
```

Same thing again, more `NS` name server records. This is the
`c.gtld-servers.net` name server telling us, I don't know the IP for
`example.com`, but here are some name servers that might!"

So we try again:

```
dig @a.iana-servers.net example.com
```

And at last we get the `A` record with the IP!

```
example.com.            86400   IN      A       93.184.216.34
```

You can also use `+trace` on the command line to watch the entire query
from start to end:

```
dig +trace example.com
```

## What to Do

Try to answer the following:

* What's the IP address of `microsoft.com`?
* What's the mail exchange for `google.com`?
* What are the name servers for `duckduckgo.com`?
* Following the process in the Digging from a Root Name Server in the
  section above, start with a root name server and dig your way down to
  `www.yahoo.com` (**NOT** `yahoo.com`).

  Note that this ends in a `CNAME` record! You'll have to repeat the
  process with the alias named by the `CNAME` record starting from the
  root servers again.

  Add to your document the `dig` commands you used to get the IP
  address. Each `dig` command should be `@` a different name server,
  starting with the root.

<!-- Rubric

5
Submission properly shows IP of www.oregonstate.edu

5
Submission properly shows MX for google.com

5
Submission properly shows name servers for oregonstate.edu

15
Submission properly shows all dig queries to get the IP address of www.yahoo.com

-->
