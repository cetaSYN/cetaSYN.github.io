---
title: "Linux Desktop Not Pulling An IPv6 Address"
date: 2020-05-01 19:29:05 -0500
classes: wide
categories:
  - linux
tags:
  - linux
  - networking
  - troubleshooting
  - ipv6
  - dhcpv6
---

Configure your host firewall to allow DHCPv6 inbound.

## Scenario

I'm going to set up a scenario here for you. Half to describe the situation in which I encountered this issue, but also half so that this hopefully registers some words that strike well with the great search engine in the sky, directing those in similar peril to the resolution. **You can probably skip to [Problem](#problem) if you were searching for this**

In my home network I use pfSense as my gatway, router, DHCP server, etc. Recently, I decided to set up IPv6 on the LAN side. There's a lot that goes into this, and for reasons I was forced to use ULA addressing because of my ISP. All of that is unrelated though and a long story for another time. Long story short, I decided to set up my pfSense using ULA addresses and `radvd` or `DHCPv6 Server` in `Assisted` mode, allowing hosts to decide whether or not they wanted to determine their own address, and to give configuration options regardless of that decision. Then came the problem.

## Problem

After configuring `radvd`/`DHCPv6 Server` on pfSense, most devices were working fine, with the exception of my Linux Desktop, which was running Ubuntu 18.04.

## Solution

Configure your host firewall to allow DHCPv6 inbound.

For my Ubuntu box, it looked like this:
{% highlight bash %}
ip6tables -A INPUT -m state --state NEW -p udp --dport 546 -d fe80::/10 -j ACCEPT
{% endhighlight %}

There's more things you should whitelist, but this will get the gears turning.

You might be asking yourself, "Now wait a minute, I didn't have to specifically whitelist DHCP for IPv4. What gives?"
Dunno.
I did some Googling and I'm lead to believe it has something to do with dhclient using raw sockets for v4 but standard UDP sockets for v6.
I haven't confirmed this with any official documentation though, and that's not what you're here for.

Enjoy your fresh new addresses

## References

- <https://tools.ietf.org/html/rfc4291#section-2.4> - Address Type Identification : IPv6 Addressing Architecture
- <https://tools.ietf.org/html/rfc8415#section-7)> - DHCP Constants : DHCPv6