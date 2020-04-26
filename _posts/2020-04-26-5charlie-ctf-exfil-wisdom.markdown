---
layout: post
title:  "5Charlie CTF - Exfil - Wisdom"
date:   2020-04-26 13:18:45 -0500
categories: [ctf, write-up, network, covert-channel]
---

## Wisdom 1

### Wisdom 1 - Challenge

There's a lot of noise in the attached PCAP, and somewhere in there, a priceless piece of wisdom.

What IP address received the most traffic?

**Attachments:** `exfil1.pcap`

### Wisdom 1 - Solution

This boils down to knowing your tools.

Wireshark > Statistics > Converations > IPv4, sort by "Bytes A->B"

![Wireshark statistics sorted by traffic bytes](/assets/images/exfil_wisdom_statistics.png)

**Flag:** `192.168.1.212`

## Wisdom 2

### Wisdom 2 - Challenge

What is the flag sent to 192.168.1.210? (format: flag{some_text})

### Wisdom 2 - Solution

Looking at the traffic, it is immediately apparent that there are a lot of TCP SYNs.
Unfortunately, as interesting as this traffic is, it is a red herring.
Taking a deeper skim through the traffic, we notice that there's base64'd data in the data of some ICMP packets.

![Wireshark filtered to 192.168.1.210](/assets/images/exfil_wisdom_icmp.png)

Let's do some tshark parsing to snag the data out of there. ICMP type 8 corresponds to ICMP Echo Request (ping).

{% highlight bash %}
tshark -r exfil1.pcap \
-Y "icmp.type==8" \
-T fields -e data \
| egrep -v '^$' \
| tr -d '\n'
{% endhighlight %}

This gives us some junk with base64 sprinked in.

``` text
´F...................... !"#$%&'()*+,-./01234567ZY......QSBEYWQgam9rZSBtQSBEYWQgam9rZSBtQSBEYWQgRf......YXkgbm90IG1ha2UgYXkgbm90IG1ha2UgYXkgbm90 r......c2Vuc2UgYXQgZmlyc2Vuc2UgYXQgZmlyc2Vuc2Ugô}......c3QgYnV0IGV2ZW50c3QgYnV0IGV2ZW50c3QgYnV0P.......dWFsbHkgZmxh
Z3tdWFsbHkgZmxh
Z3tdWFsbHkgË.......0aGVfdHJ1dGhfYmV0aGVfdHJ1dGhfYmV0aGVfdHJ~£......jb21lc19hcHBhcmVjb21lc19hcHBhcmVjb21lc199°......g==
udH0uCg==
udH0uCg==
udH0uCg==
udH0uC
```

The data mixed is a repeating pattern of base64.
I'm sure there's a nice way to pull this out, but since the field lengths vary, I found it faster to just align things by hand in CyberChef.

![CyberChef analysis of a terrible pun](/assets/images/exfil_wisdom_chef.png)

Thanks, I hate it.

**Flag:** `flag{the_truth_becomes_apparent}`
