---
layout: post
title: "One Network Troubleshooting Session"
subtitle: "I outsmarted myself"
date: 2025-9-14
lang: en
ref: network-troubleshooting
permalink: /en/2025/09/14/network-troubleshooting/
tags: [macOS, network]
---

## The problem

My Mac suddenly could not reach any ZJU site over `eduroam` or over Ethernet, while everything else on the internet worked normally.

## What I did

Turned off every VPN. No change.

Wondered whether the ZJU sites were down, and ruled that out quickly with an iPhone.

Tried an iPhone hotspot to bypass the campus network. That worked.

Used `ping` and `traceroute` to follow the packets: they never reached the target, and timed out at `hop=5`.

That pointed at DNS for `learn.intl.zju.edu.cn`. Checked the DNS server list: only `8.8.8.8`.

`ping` resolved the name to `210.32.4.165`. Searching that IP for the domain matched, and the IP sits at the Yuquan campus.

So the DNS server was resolving the name to the Yuquan machine room instead of Haining. Setting the DNS server manually to the router IP fixed it.

Then I opened CFW, and it broke again. Added `*.zju.*` to CFW's bypass list. Still broken.

CFW's log showed that even in direct mode `learn.intl.zju.edu.cn` was still resolved to `210.32.4.165`. Checked the DNS server list again: it had been rewritten.

CFW's documentation: both the Mixin and TUN mode options modify the DNS server list. Turned those off, wrote the router IP back in. Fixed.

`traceroute` now resolves to an internal Haining address:

```
$ traceroute learn.intl.zju.edu.cn
traceroute to learn.intl.zju.edu.cn (10.105.7.35), 64 hops max, 40 byte packets
 1  rt-be57-0754 (192.168.2.1)  3.799 ms  2.470 ms  2.236 ms # my own router, wired to the campus network
 2  10.106.93.1 (10.106.93.1)  3.997 ms  17.770 ms  4.683 ms
 3  10.105.7.35 (10.105.7.35)  6.075 ms  3.605 ms  4.228 ms
```

## Afterwards

This trivial little thing cost me an afternoon, 4pm to 6:30pm. My skills clearly need work (lol).
