# Project: Port Scanning

We're going to do some port scanning!

**NOTE: We're only going to run this on `localhost` to avoid any legal
trouble.**

To make this happen, we need to install the `nmap` too.

MacOS:

```
brew install nmap
```

Windows WSL:

```
sudo apt-get update
sudo apt-get install nmap
```

## 1. Portscan All Common Ports

This command will portscan 1000 of the most common ports:

```
nmap localhost
```

What's the output?

## 2. Portscan All Ports

This will scan all the ports--starting from `0` on:

```
nmap -p0- localhost
```

What's the output?

## 3. Run a Server and Portscan

Run any TCP server program that you wrote this quarter on some port.

Run the all-port scan again (above).

* Notice your server's port in the output!

* Does your server crash with a "Connection reset" error? If so, why? If
  not, speculate on why this might happen even if you didn't see it from
  your server. (See the Exploration for this!)

<!-- Rubric

20 points

5
Appropriate output from `nmap localhost`

5
Appropriate output from `nmap -p0- localhost`

5
Appropriate output from `nmap -p0- localhost` that shows your server's open port

5
Explanation of the possible "Connection reset" error

-->
