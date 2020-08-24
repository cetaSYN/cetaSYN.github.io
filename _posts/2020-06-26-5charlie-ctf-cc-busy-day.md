---
title: "5Charlie CTF - Covert-Channel - Busy Day"
date: 2020-06-26 17:00:00 -0500
classes: wide
categories:
  - write-up
tags:
  - ctf
  - write-up
  - network
  - covert-channel
---

A write-up of the "Busy Day" network covert-channel pcap analysis challenge from 5Charlie CTF.

## Busy Day

### Busy Day - Challenge

Just a normal day at the office. Get to work looking at that useless traffic!

**Attachments:** `traffic.pcapng` - Packet Capture

### Busy Day - Solution

Opening the packet catpure we see a lot of traffic, most of it unrelated to the task at hand.
It took me a while, but I eventually noticed a TCP RST that didn't have any preceding traffic.

![Suspicious TCP RST](/assets/images/busy_rst.png)

I filter on the destination IP address and, bingo, that's lookin' awfully strange.

![Wireshark filtering packets by detination address](/assets/images/busy_filtered.png)

After the filter is applied, I like to hold the up and down arrow key for a moment to scroll back and forth and get an idea of which fields are changing between the packets.
I notice the TCP sequence field is changing, which is normal.
However, they're not changing sequentially and I notice that the first byte starts spelling out a letter of "flag" for each packet.

We can then write up a quick tshark and inline-python one-liner to pull our flag.

{% highlight bash %}
tshark -r traffic.pcapng \
-Y "tcp.stream == 25 && ip.dst==104.236.250.248" \
-T fields \
-e tcp.seq_raw \
| python3 -c "import sys; [print(chr(int(str(hex(int(d)))[2:4], base=16)), end='') for d in sys.stdin.readlines()]; print('\n')"
{% endhighlight %}

**Flag:** `flag{BouncinPackets!}`
