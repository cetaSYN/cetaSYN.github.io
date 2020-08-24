---
title: "5Charlie CTF - Exfil - MD5"
date: 2020-04-26 14:15:21 -0500
classes: wide
categories:
  - write-up
tags:
  - ctf
  - write-up
  - network
  - covert-channel
---

A write-up of the "MD5" network covert-channel pcap analysis challenge from 5Charlie CTF.

## MD5

### MD5 - Challenge

A file was exfiltrated over an unencrypted protocol. What is the MD5 hash of the exfiltrated file?

**Attachments:** `net_analysis_Exfil_2-DNS_dns2.pcap`

### MD5 - Solution

This is a small pcap and most of the traffic is going to `192.168.1.210`.
Let's have Wireshark focus in on that traffic and look for anything strange.

![Wireshark filtered to 192.168.1.210](/assets/images/exfil_md5_dns.png)

That's some interesting DNS traffic. I'd be willing to bet the file is in those DNS subdomains. Let's pull it out with some tshark parsing.

{% highlight bash %}
tshark -r net_analysis_Exfil_2_-_DNS_dns2.pcap \
-Y 'dns.qry.name matches ".*\.5charlie" && ip.dst == 192.168.1.4 && !icmp' \
-T fields -e dns.qry.name \
| cut -d'.' -f1
{% endhighlight %}

I also appended `| xclip -selection clip` to the end so that I could paste it right from my clipboard into CyberChef.

![CyberChef rendering of image](/assets/images/exfil_md5_chef.png)

CyberChef immediately identifies and suggests to convert it from Hex and render as an image.

Adding the "MD5" recipe to CyberChef gives us our flag.

**Flag:** `25b15de143f92f45813a67726b959f2e`
