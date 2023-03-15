# Network Hardware

In this chapter we'll take a look at a number of hardware networking
components.

Some of these you have at home or on your computer already.

## Terminology and Components

* **Network Topology**: describes the layout of a network, how devices
  and nodes are connected, and how data flows from one part of the
  network to another.

* **Mbs**: Megabits per second (compare to "MBs", megabytes per second).

* **Gbs**: Gigabits per second, 1000 Mbs (compare to "GBs", gigabytes per second).

* **Bandwidth**: Measured in bits per second, how much data a certain
  type of cable can carry over a certain amount of time. e.g. 500 Mbs.

* **ISP**: Internet Service Provider, the company that you pay for your
  Internet connection. (Or that provides it, even if you aren't paying!)

* **Twisted Pair Cable**: What people typically think of as "Ethernet
  cable". It's a cable with plug-in jacks on either end. Internally,
  pairs of wires are twisted together to reduce interference. These
  cables have published maximum length "runs" that determine how far you
  can string a cable before you run into trouble, usually 50-100 meters
  depending on the cable and traffic speed.
  * **10baseT**: 10 Mbs twisted-pair for Ethernet
  * **100baseTX**: 100 Mbs twisted-pair for _Fast Ethernet_.
  * **1000baseT**: 1 Gbs twisted-pair for _Gigabit Ethernet_.
  * **10GbaseT**: 10 Gbs twisted-pair for _10 Gigabit Ethernet_.
  * **Category-5**: Called _cat-5_ for short, a twisted pair wire built
    to category-5 specs. Good for Fast Ethernet.
  * **Category-6**: Called _cat-6_ for short, a twisted pair wire built
    to category-6 specs. Good for Gigabit Ethernet.

* **Network Port**: Not to be confused with TCP or UDP port numbers,
  which are completely different, in this context refers to a physical
  socket on a device where you can plug in a network cable.

* **Ethernet Cable**: Conversational term meaning a twisted pair cable
  that you plug into Ethernet devices.

* **Crossover Cable**: Cable where the transmit and receive pins are
  swapped on one end of the cable. This is typically used when
  connecting two devices directly to each other when normally a switch
  or hub would be used as an intermediary. If you're plugging one
  computer directly into another with an Ethernet cable, it should
  probably be a crossover cable. Opposite of _straight-through_ cable.

* **Auto-sensing**: A network port that can tell if it has a
  straight-through or cross-over cable plugged into it, and can reverse
  the transmit and receive signals if it has to.

* **Thin-net/Thick-net**: Obsolete coaxial cabling used for Ethernet.

* **LAN**: Local Area Network. For Ethernet, this would be the network
  at your house. Think of a single IP subnet.

* **WAN**: Wide Area Network. Network that's not LAN. Think of a
  collection of LANs on University of company campus.

* **Dynamic IP**: This is when your IP gets set automatically with
  [DHCP](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol),
  for example. Your IP address might change over time.

* **Static IP**: This is when you hardcode the IP address for a certain
  device. The IP address doesn't change until you actively enter a new
  one.

* **Network Interface Controller (NIC)**, **Network Interface Card**,
  **Network Adapter**, **LAN Adapter**, **Network Card**: A bunch of
  different names for a hardware device that allows a computer to get on
  the network. This card might have Ethernet ports on it, or maybe it's
  just pure wireless. It's probably not even be a proper
  [card](https://en.wikipedia.org/wiki/Expansion_card) these days--maybe
  it's all on-board the same chip as a bunch of other I/O devices built
  into the motherboard.

* **Network Device (OS)**: A software device structure in the operating
  system that typically maps to a NIC. Some network devices like the
  _loopback device_ don't actually use a physical piece of hardware and
  just "transmit" data internally within the OS. On Unix-likes, these
  devices are named things like `eth0`, `en1`, `wlan2` and so on. The
  loopback device is typically called `lo`.

* **MAC address**: Media Access Control address. A unique Link Layer
  address for computers. With Ethernet, the MAC address is 6 bytes, and
  is usually written as hex bytes separated by colons:
  `12:34:56:78:9A:BC`. These address must be unique on a LAN for
  Ethernet to function correctly.

* **Hub**: A device that allows you to connect several computers via
  Ethernet cables. It'll have 4, 8, or more ports on the front. All
  these devices plugged into those ports are effectively on the same
  "wire" once connected, which is to say that any Ethernet packet
  transmitted is seen by **all** devices plugged into the HUB. You don't
  typically see these any longer, since switches perform the same role
  better.

* **Switch**: A hub with some brains. It knows the MAC addresses on the
  other side of the ports so it doesn't have to retransmit Ethernet
  packets to _everyone_. It just sends them down the proper wire to the
  correct destination. Help prevent network overload.

* **Router**: A Network Layer device that have multiple interfaces and
  chooses the correct one to send traffic down so that it will
  eventually reach its destination. Routers contain _routing tables_
  that let them decide where to forward a packet with a given IP
  address.

* **Default Gateway**: A router that handles traffic to all other
  destinations, if a specific route to the destination isn't known. A
  computer's routing table specifies the default gateway. On a home LAN,
  this is the IP of the "router" the ISP gave you.

  Imagine an island with a small town on it. The island is connected to
  the mainland by a single bridge. If someone wants to know where to go
  in town, you give them directions in town. For **all other
  destinations**, they drive across the bridge. In this analogy, the
  bridge is the default gateway for traffic.

* **Broadcast**: To send traffic to everyone on the LAN. This can be
  done at the Link Layer by sending an Ethernet frame to MAC address
  `ff:ff:ff:ff:ff`, or at the Network Layer by sending an IP packet with
  all the host bits set to `1`. For instance, if the network is
  `192.168.0.0/16`, the broadcast address would be `192.168.255.255`.
  You can also broadcast to `255.255.255.255` for the same effect.
  IP routers do not forward IP broadcast packets, so they are always
  restricted to the LAN.

* **Wi-Fi**: Short for _Wireless Fidelity_ (a non-technical marketing
  trademark presumably meant to pun
  [Hi-Fi](https://en.wikipedia.org/wiki/High_fidelity)), this is your
  wireless LAN connection. Speaks Ethernet at the Link Layer. Very
  similar to using an Ethernet cable, except instead of electricity over
  copper, it uses radio waves.

* **Firewall**: A computer or device at or near where your LAN connects
  to your ISP that filters traffic, preventing unwanted traffic from
  being transmitted on your LAN. Keeps the bad guys out, hopefully.

* **NAT**: Network Address Translation. A way to have a private IP
  subnet behind the NAT device that is not visible to the rest of the
  Internet. The NAT device translates internal IP addresses and ports to
  an external IP on the router. This is why if you go to Google and ask
  "what is my ip", you'll get a different number than you'll see when
  you look at the settings on your computer. NAT is in the middle,
  translating between your internal LAN IP address and the external,
  publicly-visible IP address. We'll talk more about the details of this
  mechanism later.

* **Private Network IPv4 Addresses**: For LANs not connected to the
  Internet or LANs behind NAT, there are three common subnets that are
  used: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`. These are
  reserved for private use; no public sites will ever use them, so you
  can put them on your LAN without worrying about conflicts.

* **WiFi Modem/WiFi Router**: Loosely refers to a consumer-grade device
  that you get when you sign up with an ISP for service. Often does a
  variety of things 

* **Rack-mount**: If a device doesn't come in a nice plastic case or
  isn't meant to be plugged directly into a computer, it might be
  rack-mount. These are larger non-consumer devices like routers,
  switches, or banks of disks, that get stacked up in "racks". 

* **Upload**: Transferring a file from your local device to a remote
  device.

* **Download**: Transferring a file from a remote device to your local
  device.

* **Symmetric**: In the context of transfer speeds, means that a
  connection offers the same speeds in both directions. "1 Gbs
  symmetric" means that upload and download speeds are 1 Gbs.

* **Asymmetric**: In the context of transfer speeds, means that a
  connection offers different speeds in either direction. Usually
  written as something like "600 Mbs down, 20 Mbs up", for example,
  indicating download and upload speeds. Often shortened to "600 by 20"
  conversationally. Most general usage is people download things, not
  uploading, so companies that provide service allocate more of their
  total bandwidth on the download side.

* **Cable** (from the cable company): Many cable television companies
  offer Internet connectivity over the coaxial cable line they've run to
  your house. Speeds up to 1 Gbs aren't unheard of. Typically a
  neighborhood shares bandwidth, so your speeds will drop in the evening
  when everyone living around you is watching movies. Most cable
  offerings are asymmetric.

* **DSL**: Digital Subscriber Line. Many telephone companies offer
  Internet connectivity over the phone lines they've run to your house.
  It's slower than cable at around 100 Mbs, but bandwidth is not shared
  with neighbors. Most DSL offerings are asymmetric.

* **Fiber**: Short for _optical fiber_, uses light through glass "wires"
  instead of electricity through copper wires. Very quick. Many ISPs
  that offer relatively-cheap fiber have packages that deliver 1 Gbs
  symmetric.

* **Modem**: Short for _Modulator/Demodulator_, converts signals in one
  form to signals in another form. Historically, this meant turning
  sounds transmitted over a telephone landline into data. In modern
  usage, it means converting the signals on your Ethernet LAN to
  whatever form is needed by the ISP, e.g. cable or DSL.

* **Bridge**: A device that connects two networks at the link level,
  allowing them to behave as a single network. Some bridges blindly
  forward all traffic, other bridges are smarter about it. Cable modems
  are a type of bridge, though they often come built into the same box
  as a router and switch.

* **Vampire Tap**: Back in the old days, when you wanted to connect a
  computer to a thicknet cable, you used one of [these awesomely-named
  devices](https://en.wikipedia.org/wiki/Vampire_tap) to do it. Included
  here just for fun.

## Reflect

* What's the difference between a hub and a switch?

* What does a router do?

* Why would a router with only one network connection not make any
  sense?

* What kind of device do you connect to with your laptop where you live?
  Do you use a physical cable to connect?
  
* No need to write anything for this reflection point unless you're
  inclined, but ponder what life was like [with 300 bps
  modems](https://www.youtube.com/watch?v=PjwnIm5Y6XE). The author's
  first modem was a
  [VICMODEM](https://www.oldcomputr.com/commodore-vicmodem-1982/),
  literally two million times slower than a modern cable connection.
  That was 40 years ago. Now imagine network speeds in the year 2062.
