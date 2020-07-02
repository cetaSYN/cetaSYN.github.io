---
title: "5Charlie CTF - MP3 - Pop"
date: 2020-04-29 19:44:15 -0500
categories:
  - write-up
tags:
  - ctf
  - write-up
  - steganography
---

A write-up of the "Pop" audio steganography challenge from 5Charlie CTF.

## MP3 - Pop

### MP3 - Pop - Challenge

One of our users just tried listening to their favorite song, but is now complaining that something is weird about it. We think it's trash regardless, but could you try to figure out what happened?

Format: flag{...}

**Attachments:** `pop.mp3`

### MP3 - Pop - Solution

First thing, we listen to the file and is sounds awful immediately. A bunch of weird chirping.
It's not so much a pattern as it is the chirping, which tells us that it's likely some data being embedded in the higher frequencies of the file to separate it from the main song.
For these sort of challenges I like to use Sonic Visualizer, specifically the spectrogram pane.

![Spectrogram visualization of pop.mp3](/assets/images/mp3_pop_visualizer.png)

**Flag:** `flag{1Forrest1}`
