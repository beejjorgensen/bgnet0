# Domain Name System (DNS)

We've learned that IP is responsible for routing traffic around the
Internet. We've also learned that it does it with _IP addresses_, which
we commonly show in dots-and-numbers format for IPv4, such as
`10.1.2.3`.

But as humans, we rarely use IP addresses. When you use your web
browser, you don't typically put an IP address in the address bar.

Even in our projects, we tended to type `localhost` instead of our
localhost address of `127.0.0.1`.

The general process for converting a name like `www.example.com` that
humans use into an IP address that computers use is called _domain name
resolution_, and it provided by a distributed group of servers that
comprise the _Domain Name System_, or DNS.

## Typical Usage

From a user standpoint, we configure our devices to have a "name server"
that they contact to convert those names to IP addresses. (Probably this
is configured with DHCP, but more on that later.)

When you try to connect to `example.com`, your computer contacts that
name server to get the IP address.

If that name server knows the answer, it supplies it. But if it doesn't,
then a whole bunch of wheels get put into motion.

Let's start diving down into that process.

## Domains and IP Addresses

If haven't ever registered a domain (such as `example.com` or
`oregonstate.edu` or `google.com` or `army.mil`), the process is
something like this:

1. Contact a _domain registrar_ (i.e. some company that has the
   authority to sell domains).
2. Choose a domain no one has picked yet.
3. Pay them some money annually to use the domain.
4. ...
5. Profit!

But doing this is completely disconnected from the idea of an IP
address. Indeed, domains can exist without IP addresses--they just can't
be used.

Once you have your domain, you can contact a hosting company that will
provide you with an IP address on a server that you can use.

Now you have the two pieces: the domain and the IP address.

But you still have to connect them so people can look up your IP if they
have your domain name.

To do this, you add a database record with the pertinent information to
a server that's part of the DNS landscape: a _domain name server_.

## (Domain) Name Servers

Usually called _name servers_ for short, these servers contain IP
records for the domain in which they are an _authority_. That is, a name
server doesn't have records for the entire world; it just has them for a
particular domain or subdomain.

> A _subdomain_ is a domain administered by the owner of a domain. For
> example, the owner of `example.com` might make subdomains
> `sub1.example.com` and `sub2.example.com`. These aren't hosts in this
> case--but they can have their own hosts, e.g.
> `host1.sub1.examople.com`, `host2.sub1.example.com`,
> `somecompy.sub2.example.com`.
>
> Domain owners can make as many subdomains as they want. They just have
> to make sure they have a name server set up to handle them.

A name server that's authoritative for a specific domain can be asked
about any host on that domain.

The host is often the first "word" of a domain name, though it's not
necessarily.

For example with `www.example.com`, the host is a computer called `www`
on a domain `example.com`.

A single name server might be authoritative for many domains.

But even if a name server doesn't know the IP address of the domain it's
been asked to provide, it can contact some other name servers to figure
it out. From a user perspective, this process is transparent.

So, easy-peasy. If I don't know the domain in question, I'll just
contact the name server for that domain and get the answer from them,
right?

## Root Name Servers

We have an issue, though. How can I connect to the name server for a
domain if I don't know what the name server is for a domain?

To solve this, we have a number of _root name servers_ that can help us
on our way. When we don't know an IP, we can start with them and ask
them to tell us IP, or tell us which other server to ask. More on that
process in a minute.

Computers are preconfigured with the IP addresses of the 13 root name
servers. These IPs rarely ever change, and only one of them is needed to
work. Computers that perform DNS frequently retrieve the list to keep it
up to date.

The root name servers themselves are named `a` to `m`:

``` {.default}
a.root-servers.net
b.root-servers.net
c.root-servers.net
...
k.root-servers.net
l.root-servers.net
m.root-servers.net
```

## Example Run

Let's start by doing a query on a computer called
`www.example.com`. We need to know its IP address. We don't know
which name server is responsible for the `example.com` domain. All we
know is our list of root name servers.

1. Let's choose a random root server, say `c.root-servers.net`. We'll
   contact it and say, "Hey, we're looking for `www.example.com`. Can
   you help us?"

   But the root name server doesn't know that. It says, "I don't know
   about that, but I can tell you if you're looking for any `.com`
   domain, you can contact any one of these name servers." It attaches a
   list of name servers who know about the `.com` domains:

   ``` {.default}
   a.gtld-servers.net
   b.gtld-servers.net
   c.gtld-servers.net
   d.gtld-servers.net
   e.gtld-servers.net
   f.gtld-servers.net
   g.gtld-servers.net
   h.gtld-servers.net
   i.gtld-servers.net
   j.gtld-servers.net
   k.gtld-servers.net
   l.gtld-servers.net
   m.gtld-servers.net
   ```

2. So we choose one of the `.com` name servers.

   "Hey `h.gtld-servers.net`, we're looking for `www.example.com`. Can
   you help us?"

   And it answers, "I don't know that name, but I do know the name
   servers for `example.com`. You can talk to one of them. It attaches
   the list of name servers who know about the `example.com` domain:

   ``` {.default}
   a.iana-servers.net
   b.iana-servers.net
   ```

3. So we choose one of those servers.

   "Hey `a.iana-servers.net`, we're looking for `www.example.com`. Can
   you help us?"

   And that name server answers, "Yes, I can! I know that name! Its IP
   address is `93.184.216.34`!"

So for any lookup, we start with root name server and it directs us on
where to go to find more info. (Unless the information has been cached
somewhere, but more on that later.)

## Zones

The Domain Name System is split into logical administrative _zones_. A
zone is, loosely, a collection of domains under the authority of a
particular name server.

But that's an oversimplification. There could be one or more domains in
the same zone. And there could be a number of name servers working in
that same zone.

Think of the zones as all the domains and subdomains some administration
is responsible for.

For example, in the root zone, we saw there were a number of name
servers responsible for that lookup. And also in the `.com` zone, there
were a number of different name servers there with authority.

## Resolver Library

When you write software that uses domain names, it calls a library to do
the DNS lookup. You might have noticed that in Python when you called:

``` {.py}
s.connect(("example.com", 80))
```

you didn't have to worry about DNS at all. Behind the scenes, Python did
all that work of looking up that domain in DNS.

In C, there's a function called `getaddrinfo()` that does the same
thing.

The short of it is that there's a library that we can use and we don't
have to write all that code ourselves.

The OS also has a record containing its default name server to use for
lookups. (This is sometimes configured by hand, but more commonly is
configured through
[DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol).)
So when you request a lookup, your computer first goes to this server.

But wait a minute--how does that tie into the whole root server
hierarchy thing?

The answer: caching!

## Caching Servers

Imagine all the DNS lookups that are happening globally. If we had to go
to the root servers for _every_ request, not only would that take a long
time for repeated requests, but the root servers would get absolutely
hammered.

To avoid this, all DNS resolver libraries and DNS servers _cache_ their
results.

> As it is, the root servers handle literally trillions of requests per
> day.

This way we can avoid overloading the root servers with repeated
requests.

So we're going to have to amend the outline we already went over.

1. Ask our resolver library for the IP address. If it has it cached, it
   will return it.

2. If it doesn't have it, ask our local name server for the IP address.
   If it has it cached, it will return it.

3. If it's not cached **and** if this name server has another upstream
   name server, it asks that name server for the answer.

4. If it's not cached **and** if this name server does not have another upstream
   name server, it goes to the root servers and the process continues as
   before.

With all these possible opportunities to get a cached result, it really
helps take the load off the root name servers./

Lots of WiFi routers you get also run caching name servers. So when DHCP
configures your computer, your computer uses your router as a DNS
server for the computers on your LAN. This gives you a snappy response
for DNS lookups since you have a really short ping time to your router.

### Time To Live

Since the IP address for a domain or host might change, we have to have
a way to expire cache entries.

This is done through a field in the DNS record called _time to live_
(TTL). This is the number of seconds a server should cache the results.
It's commonly set to 86400 seconds (1 day), but could be more or less
depending on how often a zone administrator thinks an IP address will
change.

When a cache entry expires, the name server will have to once again ask
for the data from upstream or the root servers if someone requests it.

## Record Types

So far, we've talked about using DNS to map a host or domain name to an
IP address. This is one of the types of records stored for a domain on a
DNS server.

The common record types are:

* `A`: An address record for IPv4. This is the type of record we've been
  talking about this whole time. Answers the question, "What is the IPv4
  address for this host or domain?"

* `AAAA`: An address record for IPv6. Answers the question, "What is the
  IPv6 address for this host or domain?"

* `NS`: A name server record for a particular domain. Answers the
  question, "What are the name servers answering for this host or
  domain?"

* `MX`: A mail exchange record. Answers the question, "What computers
  are responsible for handling mail on this domain?"

* `TXT`: A text record. Holds free-form text information. Is sometimes
  used for anti-spam purposes and proof-of-ownership of a domain.

* `CNAME`: A canonical name record. Think of this as an alias. Makes
  the statement, "Domain xyz.example.com is an alias for
  abc.example.com."

* `SOA`: A start of authority record. This contains information about a
  domain, including its main name server and contact information.

There are [a lot of DNS record
types](https://en.wikipedia.org/wiki/List_of_DNS_record_types).

## Dynamic DNS

Typical users of the Internet don't have a _static IP address_ (that is,
dedicated or unchanging) at their house. If they reboot their modem,
their ISP might hand them a different IP address.

This causes a ruckus with DNS because any DNS records pointing to their
public IPs would be out of date.

Dynamic DNS (DDNS) aims to solve this problem.

In a nutshell, there are two mechanisms at play:

1. A way for a client to tell the DDNS server what their IP address is.

2. A very short TTL on the DDNS server for that record.

While DNS defines a way to send update records, a common other way is
for a computer on your LAN to periodically (e.g. every 10 minutes)
contact the DDNS provider with an authenticated HTTP request. The DDNS
server will see the IP address it came from and use that to update its
record.

## Reverse DNS

What if you have a dots-and-numbers IP address and want the host name
for that IP? You can do a _reverse DNS_ lookup.

Note that not all IP addresses have such records, and often a reverse
DNS query will come up empty.

## Reflect

* What is your name server for your computer right now? Search the net
  for how to look it up on your particular OS.

* Do the root name servers know every IP address in the world?

* Why would anyone use dynamic DNS?

* What is TTL used for?

