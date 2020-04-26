---
layout: post
title:  "5Charlie CTF - Covert Channel - Normal"
date:   2020-04-25 22:36:40 -0500
categories: [ctf, write-up, network, covert-channel]
---

## Normal

### Normal - Challenge

Just some normal traffic. Can you find a flag?

**Attachments:** `normal.pcapng`

### Normal - Solution

Taking a look at the pcap most things look normal at a glance with one exception. There are intermittent TCP SYNs to port 8000 sprinkled in.
We can confirm the funky traffic by looking at the TCP conversations in Wireshark, sorted by destination port.

![Wireshark TCP Conversations](/assets/images/covert_normal_wireshark_ports.png)

Filtering on our suspicious traffic, we notice that the source address changes with each request.

![Wireshark filtered by TCP port 8000](/assets/images/covert_normal_wireshark_filtered.png)

If you're familiar with how flags look in different formats, you may recognize that the offsets between the source addresses look similar to the offsets between the ordinal value of "flag". It's not an exact match though... each value is incremented by 10.

!["Flag" in various data formats](/assets/images/flag_formats.png)

Let's use `tshark` and a little scripting to pull all the data out of the last octet of each matching packet, decrementing the value returned for each by 10.

{% highlight bash %}
tshark -r normal.pcapng -Y "tcp.dstport==8000" -T fields -e ip.src_host \
| python3 -c \
"import sys;
data = sys.stdin.read();
data = data.split();
[print(int(d.split('.')[-1])-10) for d in data]"
{% endhighlight %}

Using the output we can use [CyberChef to convert it using FromDecimal](https://gchq.github.io/CyberChef/#recipe=From_Decimal('Line%20feed',false)&input=MTAyCjEwOAo5NwoxMDMKMTIzCjY2CjExNwoxMTYKODcKMTA0Cjk3CjExNgo3MwoxMDIKODQKMTA0CjEwNQoxMTUKODcKOTcKMTE1CjY1CjgyCjEwMQo5NwoxMDgKNzgKMTAxCjExNgoxMTkKMTExCjExNAoxMDcKNjMKMTI1Cg).

![CyberChef conversion of flag data](/assets/images/covert_normal_data_convert.png)

Integers go in, flag falls out. 😃

**Flag:** `flag{ButWhatIfThisWasARealNetwork?}`
