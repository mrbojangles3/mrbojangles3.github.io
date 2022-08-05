---
title: "Using ip and jq to sort statistics"
categories:
  - Blog
tags:
  - System Administration
  - Networking
  - one-liner
  - iproute2
  - jq
---
# Interface Statistics
When a linux system has many interfaces and you want to identify the interface with the highest packet or byte counters the `-s` and the `-j` flags  combined with the  `jq` utility will be of assitance.

`ip -s -j link | jq 'max_by(.stats64.rx.bytes)'` this will show the pretty print of the jq output.
```
{
  "ifindex": 2,
  "ifname": "wlp1s0",
  "flags": [
    "BROADCAST",
    "MULTICAST",
    "UP",
    "LOWER_UP"
  ],
  "mtu": 1500,
  "qdisc": "noqueue",
  "operstate": "UP",
  "linkmode": "DORMANT",
  "group": "default",
  "txqlen": 1000,
  "link_type": "ether",
  "address": "ba:ba:ba:ba:ba:ba",
  "broadcast": "ff:ff:ff:ff:ff:ff",
  "stats64": {
    "rx": {
      "bytes": 1049995463,
      "packets": 819829,
      "errors": 0,
      "dropped": 10963,
      "over_errors": 0,
      "multicast": 0
    },
    "tx": {
      "bytes": 42628714,
      "packets": 200952,
      "errors": 0,
      "dropped": 0,
      "carrier_errors": 0,
      "collisions": 0
    }
  }
}
```
Instead of `stats64.rx.bytes` there is also `stats64.tx.packets` or a combination of the two.
A nice option to add to the `jq` output is `ip -s -j link | jq 'max_by(.stats64.rx.bytes).ifname'` which will slot it in nicely to a script.

Also notable is the `sort_by` function of `jq`, `ip -s -j link | jq 'sort_by(.stats64.rx.bytes)[-3:]` which will give us the NICS with the top 3 highest counters.

A colleague pointed out to me that this is a static view of the counters and there could be an active NIC with incrementing counters that hasn't surpassed the highest NIC on the system.
