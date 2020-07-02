---
title: "5Charlie CTF - 0day"
date: 2020-04-25 00:01:06 -0500
categories:
- write-up
tags:
- ctf
- write-up
- forensics
- volatility
- windows
---

A write-up of the 0day Windows memory forensics challenge from 5Charlie CTF.

## 0day 1

## 0day 1 - Challenge

To access these challenges, ssh to volatility@forensics.5charlie.com using the attached private key.
A memory capture of this system is saved at `/data/0day.bin`

What is the Volatility profile for this image?

**Attachments:** `id_volatility` - SSH Key

### 0day 1 - Solution

Lets start out by asking Volatility what profile it thinks will match.

{% highlight bash %}
vol.py -f 0day.bin imageinfo
{% endhighlight %}

We can see that Volatility gives us a few different suggested profiles.

``` text
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win8SP0x64, Win81U1x64, Win2012R2x64_18340, Win2012R2x64, Win2012x64, Win8SP1x64_18340, Win8SP1x64
                     AS Layer1 : SkipDuplicatesAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/data/0day.bin)
                      PAE type : No PAE
                           DTB : 0x1aa000L
                          KDBG : 0xf80222515530L
          Number of Processors : 4
     Image Type (Service Pack) : 0
                KPCR for CPU 0 : 0xfffff80222564000L
                KPCR for CPU 1 : 0xffffd00194b60000L
                KPCR for CPU 2 : 0xffffd00190d88000L
                KPCR for CPU 3 : 0xffffd00190dc5000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2019-10-23 19:23:24 UTC+0000
     Image local date and time : 2019-10-23 15:23:24 -0400
```

I'm not an expert with Volatility so I turn to Google and find [this wonderful set of posts from Andrea Fortuna](https://www.andreafortuna.org/2017/06/25/volatility-my-own-cheatsheet-part-1-image-identification/).

Maybe we can find a difference in the number of returned processes and modules when we do a KDBG scan utilizing each of the suggested profiles.

{% highlight bash %}
for i in Win8SP0x64 Win81U1x64 Win2012R2x64_18340 Win2012R2x64 Win2012x64 Win8SP1x64_18340 Win8SP1x64; do vol.py -f 0day.bin --profile=$i kdbgscan; done
{% endhighlight %}

Well, unfortunately, all the processes and modules match.
All the profile suggestions match the requested profile.
This only helps us narrow it down to Minor Build 9600 (Windows 8.1 or Windows Server 2012 SP2).
Output truncated to relevant portions.

``` text
Volatility Foundation Volatility Framework 2.6.1
**************************************************
Instantiating KDBG using: Unnamed AS Win8SP0x64 (6.2.9200 64bit)
...
Profile suggestion (KDBGHeader): Win8SP0x64
Version64                     : 0xf80222515e58 (Major: 15, Minor: 9600)
PsActiveProcessHead           : 0xfffff8022252e300 (67 processes)
PsLoadedModuleList            : 0xfffff80222548550 (150 modules)
...

Volatility Foundation Volatility Framework 2.6.1
**************************************************
Instantiating KDBG using: Unnamed AS Win81U1x64 (6.3.17031 64bit)
...
Profile suggestion (KDBGHeader): Win81U1x64
Version64                     : 0xf80222515e58 (Major: 15, Minor: 9600)
PsActiveProcessHead           : 0xfffff8022252e300 (67 processes)
PsLoadedModuleList            : 0xfffff80222548550 (150 modules)
...

Volatility Foundation Volatility Framework 2.6.1
**************************************************
Instantiating KDBG using: Unnamed AS Win2012R2x64_18340 (6.3.9601 64bit)
Profile suggestion (KDBGHeader): Win2012R2x64_18340
Version64                     : 0xf80222515e58 (Major: 15, Minor: 9600)
PsActiveProcessHead           : 0xfffff8022252e300 (67 processes)
PsLoadedModuleList            : 0xfffff80222548550 (150 modules)
...

Volatility Foundation Volatility Framework 2.6.1
**************************************************
Instantiating KDBG using: Unnamed AS Win2012R2x64 (6.3.9601 64bit)
...
Offset (V)                    : 0xf80222515530
Profile suggestion (KDBGHeader): Win2012R2x64
Version64                     : 0xf80222515e58 (Major: 15, Minor: 9600)
PsActiveProcessHead           : 0xfffff8022252e300 (67 processes)
PsLoadedModuleList            : 0xfffff80222548550 (150 modules)
...

Volatility Foundation Volatility Framework 2.6.1
**************************************************
Instantiating KDBG using: Unnamed AS Win2012x64 (6.2.9201 64bit)
Profile suggestion (KDBGHeader): Win2012x64
Version64                     : 0xf80222515e58 (Major: 15, Minor: 9600)
PsActiveProcessHead           : 0xfffff8022252e300 (67 processes)
PsLoadedModuleList            : 0xfffff80222548550 (150 modules)
...

Volatility Foundation Volatility Framework 2.6.1
**************************************************
Instantiating KDBG using: Unnamed AS Win8SP1x64_18340 (6.3.9600 64bit)
...
Profile suggestion (KDBGHeader): Win8SP1x64_18340
Version64                     : 0xf80222515e58 (Major: 15, Minor: 9600)
PsActiveProcessHead           : 0xfffff8022252e300 (67 processes)
PsLoadedModuleList            : 0xfffff80222548550 (150 modules)
...

Volatility Foundation Volatility Framework 2.6.1
**************************************************
Instantiating KDBG using: Unnamed AS Win8SP1x64 (6.3.9600 64bit)
Profile suggestion (KDBGHeader): Win8SP1x64
Version64                     : 0xf80222515e58 (Major: 15, Minor: 9600)
PsActiveProcessHead           : 0xfffff8022252e300 (67 processes)
PsLoadedModuleList            : 0xfffff80222548550 (150 modules)
...
```

So we've narrowed our options down to `Win81U1x64, Win2012R2x64_18340, Win2012R2x64` based upon the service packs and minor version.

At this point I wsa able to guess and check submitting as the flag until `Win2012R2x64` was accepted as the solution.

I was sure to hang on to the `Offset (V): 0xf80222515530` to use for future commands to minimize the number of sanity checks when running Volatility. (Another great tip from Andrea Fortuna)

**Flag:** `Win2012R2x64`

## 0day 2

### 0day 2 - Challenge

What was the startup time for this system? Format YYYY-MM-DD HH:MM:SS

### 0day 2 - Solution

This can be solved by looking at the Start datetime in `pslist`

{% highlight bash %}
vol.py -f 0day.bin --profile=Win2012R2x64 --kdbg=0xf80222515530 pslist
{% endhighlight %}

``` text
Volatility Foundation Volatility Framework 2.6.1
Offset(V)          Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
------------------ -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0xffffe0011123e740 System                    4      0    107        0 ------      0 2019-10-17 21:36:43 UTC+0000                                 
... [truncated]
```

**Flag:** `2019-10-17 21:36:43`

## 0day 3

### 0day 3 - Challenge

What is the system’s hostname?

### 0day 3 - Solution

This can be solved by looking at the system environment variables for the `COMPUTERNAME` variable.

{% highlight bash %}
vol.py -f 0day.bin --profile=Win2012R2x64 --kdbg=0xf80222515530 envvars
{% endhighlight %}

I'll focus in on the `DumpIt.exe` process since it was run by a user rather than a service and likely has more variables.

``` text
Pid      Process              Block              Variable                       Value
-------- -------------------- ------------------ ------------------------------ -----
...[truncated]
    4948 DumpIt.exe           0x000000dc75320860 COMPUTERNAME                   WEB-SERVER-DMZ
...
```

**Flag:** `WEB-SERVER-DMZ`

## 0day 4

### 0day 4 - Challenge

What is the name of the active directory domain?

### 0day 4 - Solution

This can be solved by looking at the system environment variables for the `USERDOMAIN` variable.

{% highlight bash %}
vol.py -f 0day.bin --profile=Win2012R2x64 --kdbg=0xf80222515530 envvars
{% endhighlight %}

I'll focus in on the `DumpIt.exe` process since it was run by a user rather than a service and likely has more variables.

``` text
Pid      Process              Block              Variable                       Value
-------- -------------------- ------------------ ------------------------------ -----
...[truncated]
    4948 DumpIt.exe           0x000000dc75320860 USERDOMAIN                     area116
...
```

**Flag:** `area116`

## 0day 5

### 0day 5 - Challenge

What is the system’s IP address?

### 0day 5 - Solution

For this challenge we can take a look at the local address in the network sessions.
It will likely be an IPv4 address that is NOT a lookpack address (127.0.0.0/8) or wildcard (0.0.0.0).
We can search for network connections on Vista+ boxes using `netscan`

{% highlight bash %}
vol.py -f 0day.bin --profile=Win2012R2x64 --kdbg=0xf80222515530 envvars
{% endhighlight %}

``` text
Offset(P)          Proto    Local Address                  Foreign Address      State            Pid      Owner          Created
0x80627490         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80627490         UDPv6    :::0                           *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80627ba0         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80627ba0         UDPv6    :::0                           *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x806281f0         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x806281f0         UDPv6    :::0                           *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80628900         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80628900         UDPv6    :::0                           *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80629660         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80629660         UDPv6    :::0                           *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80629ba0         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x80629ba0         UDPv6    :::0                           *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x8062a490         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x8062a490         UDPv6    :::0                           *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x8062ad70         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x8062ad70         UDPv6    :::0                           *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
0x8062b660         UDPv4    0.0.0.0:0                      *:*                                   796      csrss.exe      2019-10-23 14:25:02 UTC+0000
...[truncated, but more of the same tens of thousands of times]
```

That's weird. I don't know what would cause Volatility to find so many wildcard UDP binds.
It's not relevant to our task, so I'll just negate the repeating pattern using `egrep -v "(UDPv4|UDPv6) {4}(0.0.0.0|::):0"`

{% highlight bash %}
vol.py -f 0day.bin --profile=Win2012R2x64 --kdbg=0xf80222515530 envvars | egrep -v "(UDPv4|UDPv6) {4}(0.0.0.0|::):0"
{% endhighlight %}

``` text
Offset(P)          Proto    Local Address                  Foreign Address      State            Pid      Owner          Created
...[truncated]
0x848349f0         TCPv4    172.116.5.105:52856            172.116.4.4:135      CLOSED           500      lsass.exe      
0x84c7d9f0         TCPv4    172.116.5.105:135              192.168.3.71:19794   CLOSED           612      svchost.exe    
0x852279f0         TCPv4    172.116.5.105:445              192.168.3.16:57010   ESTABLISHED      4        System         
0x8563f300         TCPv4    172.116.5.105:49154            192.168.3.71:31668   ESTABLISHED      816      svchost.exe    
0x857c4300         TCPv4    172.116.5.105:52873            172.116.4.4:88       CLOSED           500      lsass.exe      
0x85a389f0         TCPv4    172.116.5.105:135              192.168.3.49:37492   CLOSED           612      svchost.exe
...
```

We can see that the local address `172.116.5.105` has been used multiple times.

**Flag:** `172.116.5.105`

## 0day 6

### 0day 6 - Challenge

What SID proves session 4 is granted domain admin rights?

### 0day 6 - Solution

This can be determined by cross-referencing `sessions` to `getsids`.

{% highlight bash %}
vol.py -f 0day.bin --profile=Win2012R2x64 --kdbg=0xf80222515530 sessions
{% endhighlight %}

Here we see a few PIDs associated with session 4. Let's make note of these. I'm gonna focus in on 4220 (iexplore.exe).

``` text
...[truncated]
**************************************************
Session(V): ffffd00191135000 ID: 4 Processes: 10
PagedPoolStart: fffff90140000000 PagedPoolEnd fffff9213fffffff
 Process: 796 csrss.exe 2019-10-23 14:44:33 UTC+0000
 Process: 4120 winlogon.exe 2019-10-23 14:44:33 UTC+0000
 Process: 1916 dwm.exe 2019-10-23 14:44:33 UTC+0000
 Process: 4832 taskhostex.exe 2019-10-23 15:04:25 UTC+0000
 Process: 3036 explorer.exe 2019-10-23 15:04:25 UTC+0000
 Process: 5076 vmtoolsd.exe 2019-10-23 15:04:35 UTC+0000
 Process: 4080 iexplore.exe 2019-10-23 15:05:00 UTC+0000
 Process: 4220 iexplore.exe 2019-10-23 15:05:00 UTC+0000
 Process: 416 mmc.exe 2019-10-23 16:01:35 UTC+0000
 Process: 1356 LogonUI.exe 2019-10-23 16:22:26 UTC+0000
```

Now to get the security identifiers (SID).

{% highlight bash %}
vol.py -f 0day.bin --profile=Win2012R2x64 --kdbg=0xf80222515530 getsids
{% endhighlight %}

We could grep these to the PID we're looking for, but I didn't do so this time.

``` text
...[truncated]
iexplore.exe (4220): S-1-5-21-4092088994-1057394591-2624646455-1491
iexplore.exe (4220): S-1-5-21-4092088994-1057394591-2624646455-513 (Domain Users)
iexplore.exe (4220): S-1-1-0 (Everyone)
iexplore.exe (4220): S-1-5-32-545 (Users)
iexplore.exe (4220): S-1-5-32-544 (Administrators)
iexplore.exe (4220): S-1-5-4 (Interactive)
iexplore.exe (4220): S-1-2-1 (Console Logon (Users who are logged onto the physical console))
iexplore.exe (4220): S-1-5-11 (Authenticated Users)
iexplore.exe (4220): S-1-5-15 (This Organization)
iexplore.exe (4220): S-1-5-5-0-123049355 (Logon Session)
iexplore.exe (4220): S-1-2-0 (Local (Users with the ability to log in locally))
iexplore.exe (4220): S-1-5-21-4092088994-1057394591-2624646455-512 (Domain Admins)
iexplore.exe (4220): S-1-18-1 (Authentication Authority Asserted Identity)
iexplore.exe (4220): S-1-5-21-4092088994-1057394591-2624646455-572
iexplore.exe (4220): S-1-16-4096 (Low Mandatory Level)
...
```

There we go, `iexplore.exe (4220): S-1-5-21-4092088994-1057394591-2624646455-512 (Domain Admins)`.
Internet Explorer running as Domain Admin. That's probably not a good idea.

**Flag:** `S-1-5-21-4092088994-1057394591-2624646455-512`
