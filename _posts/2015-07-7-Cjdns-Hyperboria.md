---
title: "Cjdns and Hyperboria"
categories:
  - Blog
tags:
  - Mesh Networking
---
# Cjdns Up and Running
Configuration is as simple as they say:

~~~ bash
git clone https://github.com/cjdelisle/cjdns.git
cd cjdns
./cjdroute --genconf > cjdroute.conf
# find a peer, I used a public one
# add a peer to the cjdroute.conf
sudo ./cjdroute cjdroute.conf
~~~

To simply join the network you have to add a peer to the configuration file and
launch the daemon. If you have made an error in the configuration file, it won't
be immediately obvious from the daemon, but the startup output will throw a
critical message. A handy debugging step is to generate a blank config and try
to launch the daemon with that. You should see a tun device show up in the "ip
addr show"

To become a part of then network (from your house) you would need to forward a port from your router (assuming you have one) to your machine on port given in the cjdroute.conf file.
The network is built around anonymity, so all packets are encrpyted.

## What its Like Once your On It
The first thing I noticed is that there isn't a DNS server per se. There is a
way to not have to type an entire IPv6 address into the url bar, but it can be
summarized as register with the guy who keeps track of this stuff.

## The Tun device
Cjdns creates a tun device for its network traffic. This is useful as code can
be written in user-space and not the kernel. The Tun device is given the ipv6
address.

## Routing
routing is complicated, I am still working on it, but it is similar to the old
ATM style of routing

## The Admin Interface
Every instance of cjdns has an admin interface that provides statistics for your
node

## Links
[wiki](https://wiki.projectmeshnet.org/Cjdns)
[wiki-troubleshooting](https://wiki.projectmeshnet.org/Cjdns_Troubleshooting)
[cjdns-whitepaper](https://github.com/cjdelisle/cjdns/blob/master/doc/Whitepaper.md)
