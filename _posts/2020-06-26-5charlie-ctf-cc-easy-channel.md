---
title: "5Charlie CTF - Covert-Channel - Easy Channel"
date: 2020-06-26 17:00:00 -0500
categories:
  - write-up
tags:
  - ctf
  - write-up
  - network
  - covert-channel
---

A write-up of the "Easy Channel" network covert-channel pcap analysis challenge from 5Charlie CTF.

## Easy Channel

### Easy Channel - Challenge

This one should be easy too! Add flag{...} around the flag you find in there!

**Attachments:** `easy.pcapng` - Packet Capture

### Easy Channel - Solution

Opening the packet catpure we see a lot of traffic, most of it unrelated to the task at hand.
I like to look through a few overviews, do some skimming, and also step through the traffic streams to look and see if there's anything interesting.

`Right-click on packet > Follow > TCP Stream`

Increasing the stream number (bottom right corner of the new window) we eventually (~37) start noticing single-packet TCP SYNs on port 80 with no full handshake.
We can filter on the source address and start looking in the data to see if there's anything we might be looking for.

![Wireshark filtering packets by source address](/assets/images/covert_easy_srcfilter.png)

After the filter is applied, I like to hold the up and down arrow key for a moment to scroll back and forth and get an idea of which fields are changing between the packets.
I notice the TCP identification field is changing, which is normal.
However, I've done enough CTF to recognize that the first two IDs in the series start with 66, 6c, 61, and 67.

Shameless Twitter plug, but you may recognize where I'm going here if you follow me:
![Flag Formats](/assets/images/flag_formats.png)
<https://twitter.com/CetaSyn/status/1245760646068797442>

We can then write up a quick tshark and inline-python one-liner to pull our flag.

{% highlight bash %}
tshark -r easy.pcapng \
-Y "(tcp.dstport == 80) && (ip.dst == 104.236.250.248) && tcp.window_size_value == 512" \
-T fields \
-e ip.id \
| python3 -c "import sys; [print(chr(int(d[-5:-3],base=16)), end='') for d in sys.stdin.readlines()]; print('\n')"
{% endhighlight %}

**Flag:** `flag{2Easy4Me}`
