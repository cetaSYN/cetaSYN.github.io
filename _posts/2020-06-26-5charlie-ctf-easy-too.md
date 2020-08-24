---
title: "5Charlie CTF - Easy Too"
date: 2020-06-26 17:00:00 -0500
classes: wide
categories:
  - write-up
tags:
  - ctf
  - write-up
  - reverse-engineering
  - arm
  - angr
---

A write-up of the "Easy Too" reverse engineering challenge from 5Charlie CTF.

## Easy Too

### Easy Too - Challenge

This one should be easy too! Add flag{...} around the flag you find in there!

**Attachments:** `easy_too` - Binary

### Easy Too - Solution

We start out by getting an idea of what we're diving into.

{% highlight bash %}
file easy_too
{% endhighlight %}

``` text
easy_too: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.3, for GNU/Linux 3.2.0, BuildID[sha1]=235f50417e3e316c4bcaa7ad57949a136019865f, not stripped
```

We can see we're dealing with an ARM binary, so I break out my go-to dockerized multiarch RE environment: <https://github.com/cetaSYN/CTF_Dockerfiles>.
I'll be using the `revr-multiarch` environment.

{% highlight bash %}
sudo docker run --rm -it -v"$(pwd):/root/" -w /root/ revr-multiarch
qemu-arm easy_too
{% endhighlight %}

We run the binary and are presented with a password prompt.

``` text
Enter the password: asdf
You entered: asdf
Access Denied!
```

I decided to use this opportunity to try out a new tool I learned about recently: `angr`.
Angr has the ability to simulate state and intelligently modify input to find interesting or targeted outputs.
In this case I don't know exactly what output we're looking for, but I'll throw a quick auto-solve script at it.

{% highlight python %}
#!/usr/bin/env python3
# solve.py

import angr
p = angr.Project("easy_too")
s = p.factory.entry_state(add_options=angr.options.unicorn)
sm = p.factory.simgr()
sm.run()

[
  print(i.posix.stdin.concretize())
  for i
  in sm.deadended
]
{% endhighlight %}

Give it a quick run and...

``` text
python3 solve.py
WARNING | 2020-06-25 05:50:28,056 | angr.state_plugins.symbolic_memory | The program is accessing memory or registers with an unspecified value. This could indicate unwanted behavior.
WARNING | 2020-06-25 05:50:28,056 | angr.state_plugins.symbolic_memory | angr will cope with this by generating an unconstrained symbolic variable and continuing. You can resolve this by:
WARNING | 2020-06-25 05:50:28,057 | angr.state_plugins.symbolic_memory | 1) setting a value to the initial state
WARNING | 2020-06-25 05:50:28,057 | angr.state_plugins.symbolic_memory | 2) adding the state option ZERO_FILL_UNCONSTRAINED_{MEMORY,REGISTERS}, to make unknown regions hold null
WARNING | 2020-06-25 05:50:28,057 | angr.state_plugins.symbolic_memory | 3) adding the state option SYMBOL_FILL_UNCONSTRAINED_{MEMORY_REGISTERS}, to suppress these messages.
WARNING | 2020-06-25 05:50:28,057 | angr.state_plugins.symbolic_memory | Filling register r10 with 4 unconstrained bytes referenced from 0x10794 (__libc_csu_init+0x0 in easy_too (0x10794))
... (SIMILAR REPEATED OUTPUT TRIMMED)
WARNING | 2020-06-25 05:50:41,093 | angr.engines.successors | Exit state has over 256 possible solutions. Likely unconstrained; skipping. <BV32 Reverse(packet_0_stdin_11_480[255:224])>

[b'\x01\x19P\x00p)\x00.\x01\x08\xee**\x02\x06\x02\x8a\x01J\x89\x01KL\x04\x19\x01\x02\x02\x02\x19\x02\x02*I\x02(J\x01\x8a\x00\x1b\x8a\x01*\x02\x01\x01III\x19)\x01\x08\x02\x02\x02\x02\x00\x19']
[b'\x04\x00\x02\x02\x00\x02M.\x02\x1d\x01\x96\x01\x02@\x08\x80\x10\x08\x01\x02\x08\x80\x0f\x01\x02\x80\x01\x80\x04\x08@\x80\x08\x08\x02@\x08\x08I\xab\x02\x02\x01M\x08g\x08\x10\x02\x10\x02\xa9\x04@@\x0e\x80$\x00']
[b'Eas\x08\x04\x03\x0c\x8a\x08\x00\x08\x86\x00\x19\x02\x00\x02\x01\x02\x02\x80\x02\x01\x02\x02\x82\x19\x01\x89\x02\x08\x8a\x04\x08\x02\x1a\x08\x08\x89\x08\xa8\x01J\x02\x19\x86\x89\x08\x02)J**I\x08\x00\x89\x01\x01\x00']
[b'EasyAR\x8a\x00\x00\x10\x90\x02\x00\x00\x00)\x00\x00\x10V\x00\x00\x00\x01I$\x01\x00\x00\x00\x00\x01\x80\x08\x00\x08\x03\x02\x01\x08\x01\x82\x00\x01\x00\x02*\x01\x00\x08\x02\x02I\x08\x0f\x01\x19\x02)\x8a']
[b'Eas\x02\x08JM.\x02\x02*\x1a*\x08\x0e\x02\x00(\x88\x02\x02\x1a\x08\x02J\x02I8\x01\x012\x82\x02j\x01I\x0cI\x01*\x19\x02\x02\x02\x01II\x8bJ\x1a\x01\x08D\x10\x00\x89\x02I\x02\x01']
[b'EasyARM.\x01\x02\x08\x00\x00\x02\x02\x01\x89\x02\x02\x89\x19\x8aI\x01\x02\x01J\x1a\x89\x08\x01\x01)\x01\x8a)\x01\x0e\x02J\x08\x89\x01\x08\x01\x0c\x01\x08\x19*\x02\x01\x01\x01\x01I\x02\x08II']
```

These 6 binary strings are the inputs that angr found causing different end states.
We see `EasyARM.` in the beginning of the last index, which is likely our flag.
It is likely that the binary only evaluates up to a certain amount of characters, so the trailing characters are junk that can be discarded.
Let's validate our flag...

``` text
Enter the password: EasyARM.
You entered: EasyARM.
Access Granted!

```

Staring at assembly is great, but wow, angr is a great tool.

**Flag:** `flag{EasyARM.}`
