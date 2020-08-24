---
title: "5Charlie CTF - BMP"
date: 2020-04-25 18:47:03 -0500
classes: wide
categories:
- write-up
tags:
- ctf
- write-up
- steganography
---

A write-up of the BMP image steganography challenge from 5Charlie CTF.

## BMP

### BMP 1 - Challenge

In the attached image, what is the byte offset of the file section where the pixel array is stored? (Give answer in format: 0xBC)

**Attachments:**

![`stego_Image_2_-_bmp_COG.bmp`](/assets/images/stego_Image_2_-_bmp_COG.bmp)

### BMP 1 - Solution

This mostly boils down to understanding/researching file formats.
During the CTF I used `xxd` but for the sake of this write-up I'll use `wxHexEditor` to show what I mean more clearly.

We start by finding out what the fields match up to

![`bmp_file_headers_wikipedia.png`](/assets/images/bmp_file_headers_wikipedia.png)
<https://en.wikipedia.org/wiki/BMP_file_format>

Now we can start mapping out fields to figure out which bytes we're looking for.

![`stego_Image_2_-_bmp_COG_wxHexeditor`](/assets/images/stego_Image_2_-_bmp_COG_wxHexeditor.png)

The green field is the offset we're looking for.

I used Python to convert it to decimal because I'm lazy and that info lets me easily mark the data for the pink image data section.

Since it's little-endian, we'll read the bytes from right to left for our flag and drop the leading zeros.

**Flag:** `flag{0x436}`

### BMP 2 - Challenge

Find the flag hidden in COG.bmp. (Answer will be in format: flag{some_text})

### BMP 2 - Solution

Moving to finding the flag, since we weren't provided any sort of riddle or additional information, it's unlikely the flag is encrypted or offset in any unique way that would require special processing.

I started this section by refreshing my memory on what patterns I would be looking for.

!["flag" in various data formats](/assets/images/flag_formats.png)

Since we don't see any of the characters of "flag" in a dump of the strings, we're probably going to be looking for the binary formatted flag.

"fl" in particular makes for a fun pattern: `0110 0110 0110 1100`

Let's pop the file open with `xxd -b stego_Image_2_-_bmp_COG.bmp | less`:

![`stego_Image_2_-_bmp_COG_xxd`](/assets/images/stego_Image_2_-_bmp_COG_xxd.png)

The pattern is easy to spot after a little looking.

Now to pull the rest of the flag with a little bit of scripting.

{% highlight bash %}
xxd -b bmp_cut.hex \
| cut -d' ' -f 2-7 \
| python3 -c \
"import sys; data = sys.stdin.read();
data = data.split();
print(''.join([''.join([d[i] for i in range(8) if i==7])
for d in data])[24:248]);"
{% endhighlight %}

This was originally on one line but this should make it easier to read. <3

The Python slicing indexes (`[24:248]`) were determined by guess-and-check until I got the flag format I was looking for.

This dumps out a binary string that when thrown into a converter like [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Binary('None')&input=MDExMDAxMTAwMTEwMTEwMDAxMTAwMDAxMDExMDAxMTEwMTExMTAxMTAxMTAwMDAxMDEwMTExMTEwMTEwMTEwMDAxMTAxMDAxMDExMTAxMDAwMTExMDEwMDAxMTAxMTAwMDExMDAxMDEwMTAxMTExMTAxMTAwMDEwMDExMDEwMDEwMTExMDEwMDAxMDExMTExMDExMDAwMDEwMTExMDEwMDAxMDExMTExMDExMDAwMDEwMTAxMTExMTAxMTEwMTAwMDExMDEwMDEwMTEwMTEwMTAxMTAwMTAxMDExMTExMDE) gives us the flag.

**Flag:** `flag{a_little_bit_at_a_time}`
