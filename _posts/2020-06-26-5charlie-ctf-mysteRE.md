---
title: "5Charlie CTF - Myste-RE"
date: 2020-06-26 00:00:00 -0500
classes: wide
categories:
  - write-up
tags:
  - ctf
  - write-up
  - reverse-engineering
---

A write-up of the "Myste-RE" reverse-engineering challenge from 5Charlie CTF.

## Myste-RE 1

### Myste-RE 1 - Challenge

Somebody made this to do something, but what?

Note: Flag format is flag(*).

**Attachments:** `flag` - Binary

### Myste-RE 1 - Solution

This one took a hot minute to figure out.
Our team played around with various architectures and eventually agreed that 16-bit x86 made the most sense.
We were able to create environment and read through using radare2 and the commands `e asm.arch=x86` and `e asm.bits=16`.

We had some trouble getting an environment running but eventually decided to package it up as a DOSBox package and upload it to an online javascript-based DOSBox emulator: <https://caiiiycuk.github.io/dosify/>

![DOSify Input](/assets/images/mystere_dosify_input.png)

We give it a run...

![DOSify Output](/assets/images/mystere_dosify_output.png)

Now we start seeing a bunch of dots being printed to the screen.
It turns out that triple-dots are dashes, the single dots are dots, and the long breaks are character delimiters- morse code!
And then I copied that by hand into CyberChef.

<https://gchq.github.io/CyberChef/#recipe=From_Morse_Code('Space','Line%20feed')&input=LSAgLi4uLiAuCi4uLS4gLi0uLiAuLSAtLS4KLSAuLi4uIC4tIC0KLS4tLSAtLS0gLi4tCi4tIC4tLiAuCi4tLi4gLS0tIC0tLSAtLi0gLi4gLiAtLiAtLS4KLi4tLiAtLS0gLi0uCi4uIC4uLiAuLi0uIC4tLi4gLi0gLS0uIC0uLS0uIC0uLS4gLS0tLS0gLS4gLi4uLSAtLS0tLSAuLS4uIC4uLSAtIC4gLS4uIC0uLi4uLSAtLSAtLS0tLSAuLS4gLi4uIC4uLi0tIC0uLi4uLSAuLi0uIC4tLi4gLi4uLi0gLS0uIC0uLS0uLQo>

**Flag:** `FLAG(C0NV0LUTED-M0RS3-FL4G)`

Whoever made this enjoys suffering.
