# Project: Packet Tracer: Using a Switch

It's not typical to wire two PCs directly together. Usually they're
connected through a _switch_.

Let's set that up in the lab.

## Add Some PCs

Choose "End Devices" in the lower left, and drag three PCs onto the
workspace.

Click on each in turn, going to their "Config" tabs for their
`FastEthernet0` devices and giving them IP addresses:

* PC0: `192.168.0.2`
* PC1: `192.168.0.3`
* PC2: `192.168.0.4`

They should all use subnet mask `255.255.255.0`.

## Add a Switch

Click on "Network Devices" in the lower left. The bottom left row of
icons will change.

Select "Switches", second left on bottom row. The middle panel will
change.

Drag a `2960` switch onto the workspace.

If you click on the switch and look under the "Physical" tab, you'll see
the switch is a device with a _lot_ of Ethernet ports on it. (This is a
pretty high-end switch. Home switches typically have 4 or 8 ports. The
back of your WiFi router at home probably has 4 ports like this--that's
a switch built into it!)

We can connect PCs to these ports and they'll be able to talk to one
another.

## Wire It Up

None of the PCs will be directly connected. They'll all connect directly
to the switch.

We don't use crossover cables here; the switch knows what it's doing.

> Note that when you first wire up the LAN, you might not have two green
> up arrows shown on the connection. One or both of them might be orange
> circles indicating the link is in the process of coming up. You can
> hit the `>>` fast-forward button in the lower left to jump ahead in
> time until you get two green up arrows.

Choose the "Connections" selector in the lower left.

Choose the "Copper Straight-Through" cable. (The icon will change to an
"anti" symbol.)

Click on PC0, then select `FastEthernet0`.

Click on the switch, then select any `FastEthernet0` port.

Do the same for the other 2 PCs.

## Test Pings!

Click on one of the PCs and go to the "Desktop" tab and run a Command
Prompt. Make sure you can `ping` the other two PCs.

<!-- Rubric

5
Straight-through cable used

5
Three PCs used

5
Switch used

10
Can ping from any PC to any other PC

-->
