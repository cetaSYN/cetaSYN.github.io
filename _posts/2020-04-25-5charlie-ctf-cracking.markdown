---
title: "5Charlie CTF - Cracking"
date: 2020-04-25 23:42:37 -0500
categories:
  - write-up
tags:
  - ctf
  - write-up
  - cracking
---

A write-up of the "Cracking" password cracking challenge from 5Charlie CTF.

## Cracking 1

### Cracking 1 - Challenge

Manilov uses a short Python script to generate uncrackable passwords. It's a shame, because if you could crack the password you might be able to find the flag in this zip archive.

**Attachments:**  `flag.zip`, `hacking_Cracking_1_-_HashCat_password_generator.py` (Below)

{% highlight python %}
#!/usr/bin/python3
# Author: Manilov
# Purpose: My uncrackable password generator
import secrets
import string
password=secrets.choice(string.ascii_uppercase)+secrets.choice(string.punctuation)+secrets.choice(string.ascii_lowercase)+secrets.choice(string.ascii_lowercase)+secrets.choice(string.digits)+secrets.choice(string.digits)+secrets.choice(string.digits)+secrets.choice(string.ascii_lowercase)
print("{}".format(password))
{% endhighlight %}

### Cracking 1 - Solution

Examining the password generation script, it generates passwords in a predictable pattern.
This translates very well to hash cracking mask.

Let's get the hash we'll be cracking by using the `zip2john` utility.

{% highlight bash %}
./zip2john flag.zip > hash.txt
{% endhighlight %}

This gives us a hash to target.

``` text
$pkzip2$1*1*2*0*2d*21*83fe65b4*0*26*0*2d*83fe*6d72*3ac34bc56eb839b8a6d5d5559a19d653162d14529a75b76c1b9714728a6db32b001b3a8df475d0ae87dd19bc94*$/pkzip2$
```

Now we can build a mask to crack the zip password.
I prefer hashcat for cracking, so we'll be using that.
For the latest hash modes, I pull hashcat from Github and build from source.

{% highlight bash %}
hashcat -a 3 -m 17225 -O hash.txt "?u?s?l?l?d?d?d?l"
{% endhighlight %}

Here's the options for this command:

- `-a 3`: Crack using a mask attack
- `-m 17225`: Use PKZIP (Mixed Multi-File) mode
  - <https://hashcat.net/wiki/doku.php?id=example_hashes>
- `-O`: Use an OpenCL optimized kernel (helps GPU performance)
- `?u`: Uppercase letter mask
- `?s`: Specials mask (printable non-alphanumeric characters)
- `?l`: Lowercase letter mask
- `?d`: Digit mask

This gives us the password `S.tr567k`.

After unzipping the file with the password, we are given our flag.

**Flag:** `flag{strong_password_is_stronk}`
