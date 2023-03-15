# Project: Computing and Finding Subnets

In preparation for our subsequent project that finds routes across the
network, we need to do some work in figuring out how IP addresses,
subnet masks, and subnets all work together.

In this project we'll put some of the work from the chapters into
practice. We'll:

* Write functions to convert dots-and-numbers IP addresses into single
  32-bit values--and back again.

* Write a function that converts a subnet mask in slash notation into a
  single 32-bit value representing that mask.

* Write a function to see if two IP addresses are on the same subnet.

## Restrictions

You may **not** use:

* Any functionality from the `socket` module.
* Any functionality from the `struct` module.
* Any functionality from the `netaddr` module.
* The `.to_bytes()` or `.from_bytes()` methods.

Keep it in the realm of your own home-cooked bitwise operations.

## What To Do

[fls[Grab the skeleton code and other files in this ZIP
archive|netfuncs/netfuncs.zip]]. This is what you'll fill in for this
project.

Implement the following functions in `netfuncs.py`:

* `ipv4_to_value(ipv4_addr)`
* `value_to_ipv4(addr)`
* `get_subnet_mask_value(slash)`
* `ips_same_subnet(ip1, ip2, slash)`
* `get_network(ip_value, netmask)`
* `find_router_for_ip(routers, ip)`

The descriptions of the functions are in the file in their respective
docstrings. Be sure to pay special attention to the input and output
_types_ in the examples shown there.

Note that none of the functions need be more than 5-15 lines long. If
you're getting a much bigger function implementation, you might be off
track.

## Testing as you Go

I encourage you to _write one function at a time_ and test it out by
calling it with your own sample data before moving on to the next
function.

You can add your own calls to the functions to help you verify that
they're doing what they're supposed to do. Use the inputs and outputs
from the example comments for tests.

There is a function called `my_tests()` in `netfuncs.py` that will run
instead of the default main function if you uncomment it.

If you uncomment `my_tests()`, you can run the program with:

``` {.sh}
python netfuncs.py
```

and see the output from that function.

Be sure to comment out `my_tests()` and run it with the included main
code before you submit, as shown in the next section.

## Running the Program

You'll run it like this:

``` {.sh}
python netfuncs.py example1.json
```

It will read in the JSON data from the included `example1.json` and run
your functions on various parts of it.

The output, included in `example1_output.txt`, should look exactly like
this if everything is working correctly:

``` {.default}
Routers:
     10.34.166.1: netmask 255.255.255.0: network 10.34.166.0
     10.34.194.1: netmask 255.255.255.0: network 10.34.194.0
     10.34.209.1: netmask 255.255.255.0: network 10.34.209.0
     10.34.250.1: netmask 255.255.255.0: network 10.34.250.0
      10.34.46.1: netmask 255.255.255.0: network 10.34.46.0
      10.34.52.1: netmask 255.255.255.0: network 10.34.52.0
      10.34.53.1: netmask 255.255.255.0: network 10.34.53.0
      10.34.79.1: netmask 255.255.255.0: network 10.34.79.0
      10.34.91.1: netmask 255.255.255.0: network 10.34.91.0
      10.34.98.1: netmask 255.255.255.0: network 10.34.98.0

IP Pairs:
   10.34.194.188    10.34.91.252: different subnets
   10.34.209.189    10.34.91.120: different subnets
   10.34.209.229    10.34.166.26: different subnets
   10.34.250.213    10.34.91.184: different subnets
   10.34.250.228    10.34.52.119: different subnets
   10.34.250.234     10.34.46.73: different subnets
     10.34.46.25   10.34.166.228: different subnets
    10.34.52.118     10.34.91.55: different subnets
    10.34.52.158     10.34.166.1: different subnets
    10.34.52.187    10.34.52.244: same subnet
     10.34.52.23    10.34.46.130: different subnets
     10.34.52.60    10.34.46.125: different subnets
    10.34.79.218     10.34.79.58: same subnet
     10.34.79.81    10.34.46.142: different subnets
     10.34.79.99    10.34.46.205: different subnets
    10.34.91.205    10.34.53.190: different subnets
     10.34.91.68    10.34.79.122: different subnets
     10.34.91.97    10.34.46.255: different subnets
    10.34.98.184     10.34.209.6: different subnets
     10.34.98.33   10.34.166.170: different subnets

Routers and corresponding IPs:
     10.34.166.1: ['10.34.166.1', '10.34.166.170', '10.34.166.228', '10.34.166.26']
     10.34.194.1: ['10.34.194.188']
     10.34.209.1: ['10.34.209.189', '10.34.209.229', '10.34.209.6']
     10.34.250.1: ['10.34.250.213', '10.34.250.228', '10.34.250.234']
      10.34.46.1: ['10.34.46.125', '10.34.46.130', '10.34.46.142', '10.34.46.205', '10.34.46.25', '10.34.46.255', '10.34.46.73']
      10.34.52.1: ['10.34.52.118', '10.34.52.119', '10.34.52.158', '10.34.52.187', '10.34.52.23', '10.34.52.244', '10.34.52.60']
      10.34.53.1: ['10.34.53.190']
      10.34.79.1: ['10.34.79.122', '10.34.79.218', '10.34.79.58', '10.34.79.81', '10.34.79.99']
      10.34.91.1: ['10.34.91.120', '10.34.91.184', '10.34.91.205', '10.34.91.252', '10.34.91.55', '10.34.91.68', '10.34.91.97']
      10.34.98.1: ['10.34.98.184', '10.34.98.33']
```

If you're getting different output, try to look through the code and see
what functions are being used with the incorrect output. Then test those
in more detail in the `my_tests()` function.

<!--
New Rubric

5 points each, 100 points

ipv4_to_value(): returns single numeric integer type
value_to_ipv4(): returns correct string
get_subnet_mask_value(): uses bitwise operations to make mask
ipv4_to_value(): Successfully converts any IP address in dots-and-numbers format into a single value representing that IP packed into a 4-byte 32-bit integer.
value_to_ipv4(): Successfully converts a single value representing an IP packed into a 4-byte 32-bit integer into a string in dots-and-numbers format.
value_to_ipv4(): No leading zeros on any of the numbers in the string.
value_to_ipv4(): No padding--only digits and periods in the string.
get_subnet_mask_value(): Returns a single integer representing the subnet mask defined by the slash notation.
get_subnet_mask_value(): Handles both plain slash notation like "/16" and IP/slash notation like "198.51.100.12/22".
ips_same_subnet(): Returns True if both numbers are on the same subnet.
ips_same_subnet(): Uses get_subnet_mask_value() to get the subnet mask.
ips_same_subnet(): Uses ipv4_to_value() to get the values of the IP addresses.
ips_same_subnet(): Does the proper bitwise arithmetic to determine if the IP addresses are on the same subnet.
get_network(): Returns the network portion of the IP address as an integer.
get_network(): Uses the correct bitwise arithmetic to perform this computation.
find_router_for_ip(): Correctly finds the router that's on the same subnet as the given IP.
find_router_for_ip(): Returns the router IP as a dots-and-numbers strings
find_router_for_ip(): Returns None if no router is found on the same subnet as the given IP.
find_router_for_ip(): Calls ips_same_subnet() to make the determination.
No code below the do-not-modify line was modified.
-->

<!--
Fall 2022 Rubric

* `ipv4_to_value(ipv4_addr)`

10
ipv4_to_value(): Successfully converts any IP address in dots-and-numbers format into a single value representing that IP packed into a 4-byte 32-bit integer.

* `value_to_ipv4(addr)`

10
value_to_ipv4(): Successfully converts a single value representing an IP packed into a 4-byte 32-bit integer into a string in dots-and-numbers format.

1
value_to_ipv4(): No leading zeros on any of the numbers in the string.

1
value_to_ipv4(): No padding--only digits and periods in the string.

* `get_subnet_mask_value(slash)`

10
get_subnet_mask_value(): Returns a single integer representing the subnet mask defined by the slash notation.

5
get_subnet_mask_value(): Handles both plain slash notation like "/16" and IP/slash notation like "198.51.100.12/22".

* `ips_same_subnet(ip1, ip2, slash)`

10
ips_same_subnet(): Returns True if both numbers are on the same subnet.

5
ips_same_subnet(): Uses get_subnet_mask_value() to get the subnet mask.

5
ips_same_subnet(): Uses ipv4_to_value() to get the values of the IP addresses.

10
ips_same_subnet(): Does the proper bitwise arithmetic to determine if the IP addresses are on the same subnet.

* `get_network(ip_value, netmask)`

5
get_network(): Returns the network portion of the IP address as an integer.

5
get_network(): Uses the correct bitwise arithmetic to perform this computation.

* `find_router_for_ip(routers, ip)`

10
find_router_for_ip(): Correctly finds the router that's on the same subnet as the given IP.

3
find_router_for_ip(): Returns the router IP as a dots-and-numbers strings

3
find_router_for_ip(): Returns None if no router is found on the same subnet as the given IP.

5
find_router_for_ip(): Calls ips_same_subnet() to make the determination.

* Additional:

5
No code below the do-not-modify line was modified.

-->

