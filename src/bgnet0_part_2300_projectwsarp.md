
# Project: Sniff ARP packets with WireShark

We're going to take a look at some live network traffic with
[fl[WireShark|https://www.wireshark.org/]] and see if we can capture
some ARP requests and replies.

Wireshark is a great tool for _sniffing_ network packets. It gives you a
way to trace packets as that move across the LAN.

We'll set up a filter in WireShark so that we're only looking for ARP
packets to and from our specific machines so we don't have to search for
a needle in a haystack.

## What to Create

A document that contains 4 things:

1. Your MAC address of your currently active connection.

2. Your IP address of your currently active connection.

3. A human-readable WireShark packet capture of an ARP request.

4. A human-readable WireShark packet capture of an ARP reply.

Details below!

## Step by Step

Here's what we'll do:

1. **Look Up Your Ethernet (MAC) Address**

   Your computer might have multiple Ethernet interfaces (e.g. one for
   WiFi and one for wired--the Ethernet jack on the side).

   Since you're almost certainly using wireless right now, look up the
   MAC address for your wireless interface. (You might have to search
   online for how to do this.)
   
   For both this step and step 2, below, the information can be found
   with this command on Unix-likes:

   ```
   ifconfig
   ```

   and this command on Windows:

   ```
   ipconfig
   ```

2. **Look Up Your IP Address**

   Again, we want the IP address of your active network interface,
   probably your WiFi device.

3. **Launch WireShark**

   On initial launch, set up WireShark to look at your active Ethernet
   device. On Linux, this might be `wlan0`. On Mac, it could be `en0`.
   On Windows, it's likely just `Wi-Fi`.

   Set up a display filter in WireShark to filter ARP packets that are
   only either to or from your machine. Type this in the bar near the
   top of the window, just under the blue sharkfin button.

   ```
   arp and eth.addr==[your MAC address]
   ```

   Don't forget to hit `RETURN` after typing in the filter.

   Start capturing by hitting the blue sharkfin button.

4. **Find more IPs on your subnet**

   For this section it doesn't matter if there's actually a computer at
   the remote IP, but it's nice if there is. Watch the Wireshark log for
   a while to see what other IPs are active on your LAN.

   > Your IP ANDed with the subnet mask is your subnet number. Try
   > putting various numbers for the host portion. Try your default
   > gateway (search the Internet for how to find your default gateway
   > in your OS.)

   On the command line, `ping` another IP on your LAN:

   ```
   ping [IP address]
   ```

   (Hit `CONTROL-C` to break out of ping.)

   On the first ping, did you see any ARP packets go by in Wireshark? If
   not, try other IP addresses on the subnet, as noted above.

   No matter how many pings you send, you should only see one ARP
   reply. (You'll see one request per ping if there are no replies!)
   This is because after the first reply, your computer caches the ARP
   reply and no longer needs to send them out!

   After a minute or five, your computer should expire that ARP cache
   entry and you'll see another ARP exchange if you ping that IP again.

5. **Write Down the Request and Reply**

   In the timeline, the ARP request will look something like this
   excerpt (with different IP addresses, obviously):

   ```
   ARP 60 Who has 192.168.1.230? Tell 192.168.1.1
   ```

   And if all goes well, you'll have a reply that looks like this:

   ```
   ARP 42 192.168.1.230 is at ac:d1:b8:df:20:85
   ```

   [If you're not seeing anything, try changing your display filter to
   just say "arp". Watch for a while and see if you see a
   request/reply pair go by.]

   Click on the request and look at the details in the lower left panel.
   Expand the "Address Resolution Protocol (request)" panel.

   Right click any line in that panel and select "Copy->All Visible
   Items".

   Here's an example request (truncated for line length):

   ```
   Frame 221567: 42 bytes on wire (336 bits), 42 bytes captured  [...]
   Ethernet II, Src: HonHaiPr_df:20:85 (ac:d1:b8:df:20:85), Dst: [...]
   Address Resolution Protocol (request)
       Hardware type: Ethernet (1)
       Protocol type: IPv4 (0x0800)
       Hardware size: 6
       Protocol size: 4
       Opcode: request (1)
       Sender MAC address: HonHaiPr_df:20:85 (ac:d1:b8:df:20:85)
       Sender IP address: 192.168.1.230
       Target MAC address: 00:00:00_00:00:00 (00:00:00:00:00:00)
       Target IP address: 192.168.1.148
   ```

   Click on the reply in the timeline. Copy the reply information in the
   same way.

   Here's an example reply (truncated for line length):
   
   ```
   Frame 221572: 42 bytes on wire (336 bits), 42 bytes captured  [...]
   Ethernet II, Src: Apple_63:3c:ef (8c:85:90:63:3c:ef), Dst:    [...]
   Address Resolution Protocol (reply)
       Hardware type: Ethernet (1)
       Protocol type: IPv4 (0x0800)
       Hardware size: 6
       Protocol size: 4
       Opcode: reply (2)
       Sender MAC address: Apple_63:3c:ef (8c:85:90:63:3c:ef)
       Sender IP address: 192.168.1.148
       Target MAC address: HonHaiPr_df:20:85 (ac:d1:b8:df:20:85)
       Target IP address: 192.168.1.230
   ```

<!--

Rubric

10
Submission includes your MAC address of your currently active connection.

10
Submission includes your IP address of your currently active connection.

20
Submission includes a human-readable WireShark packet capture of an ARP request.

20
Submission includes a human-readable WireShark packet capture of an ARP reply.

-->
