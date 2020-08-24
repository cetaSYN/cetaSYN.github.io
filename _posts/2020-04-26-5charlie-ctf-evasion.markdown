---
title: "5Charlie CTF - Evasion"
date: 2020-04-26 12:40:13 -0500
classes: wide
categories:
  - write-up
tags:
  - ctf
  - write-up
  - linux
  - forensics
---

A write-up of the Aperture Linux live system forensics challenge from 5Charlie CTF.

## Evasion 1

### Evasion 1 - Challenge

To access these challenges ssh to mitya@analysis.5charlie.com using the attached private key.

What is the full path the backdoor executed from?

**Attachments:** `id_mitya.txt` - SSH Key

### Evasion 1 - Solution

Light digging through logs, home directories, and system configs doesn't turn up anything interesting.

Maybe the backdoor is still running?
Let's look at our /proc directory.

``` text
root@df521907feb0:/# cd /proc
root@df521907feb0:/proc# ls
1          cpuinfo      interrupts  kpagecgroup    modules       self           timer_list
12         crypto       iomem       kpagecount     mounts        slabinfo       tty
16         devices      ioports     kpageflags     mtrr          softirqs       uptime
acpi       diskstats    irq         latency_stats  net           stat           version
buddyinfo  dma          kallsyms    loadavg        pagetypeinfo  swaps          vmallocinfo
bus        driver       kcore       locks          partitions    sys            vmstat
cgroups    execdomains  key-users   mdstat         sched_debug   sysrq-trigger  xen
cmdline    filesystems  keys        meminfo        schedstat     sysvipc        zoneinfo
consoles   fs           kmsg        misc           scsi          thread-self
root@df521907feb0:/proc# ls
1          cpuinfo      interrupts  kpagecgroup    modules       self           timer_list
12         crypto       iomem       kpagecount     mounts        slabinfo       tty
17         devices      ioports     kpageflags     mtrr          softirqs       uptime
acpi       diskstats    irq         latency_stats  net           stat           version
buddyinfo  dma          kallsyms    loadavg        pagetypeinfo  swaps          vmallocinfo
bus        driver       kcore       locks          partitions    sys            vmstat
cgroups    execdomains  key-users   mdstat         sched_debug   sysrq-trigger  xen
cmdline    filesystems  keys        meminfo        schedstat     sysvipc        zoneinfo
consoles   fs           kmsg        misc           scsi          thread-self
```

It looks like we have PIDs 1 and 12 persisting, with 16/17 being a PID for the command we're currently running.
Let's look deeper into 1 and 12.

``` text
root@df521907feb0:/proc# cat 1/cmdline | tr '\0' '\n'
/bin/bash
root@df521907feb0:/proc# ls -l 1/fd/
total 0
lrwx------ 1 root root 64 Apr 26 17:52 0 -> /dev/pts/0
lrwx------ 1 root root 64 Apr 26 17:52 1 -> /dev/pts/0
lrwx------ 1 root root 64 Apr 26 17:52 2 -> /dev/pts/0
lrwx------ 1 root root 64 Apr 26 17:53 255 -> /dev/pts/0
root@df521907feb0:/proc# ls -l 1/cwd 
lrwxrwxrwx 1 root root 0 Apr 26 17:53 1/cwd -> /proc
```

Nothing too crazy there. This is probably the PID for our current shell and the parent to each command we run.

What about PID 12?

``` text
root@df521907feb0:/proc# cat 12/cmdline | tr '\0' '\n'
[kworkerd]
root@df521907feb0:/proc# ls -l 12/fd/
total 0
lr-x------ 1 root root 64 Apr 26 17:53 0 -> /dev/null
lrwx------ 1 root root 64 Apr 26 17:53 1 -> /dev/pts/0
lrwx------ 1 root root 64 Apr 26 17:53 2 -> /dev/pts/0
lrwx------ 1 root root 64 Apr 26 17:53 3 -> 'socket:[5364808]'
root@df521907feb0:/proc# ls -l 12/cwd
lrwxrwxrwx 1 root root 0 Apr 26 17:54 12/cwd -> /tmp
```

Now that looks strange. `kworkerd` etc is fairly common to see on Linux systems, but it certainly shouldn't be executing out of `/tmp` or holding open a socket.

**Flag:** `/tmp/[kworkerd]`

## Evasion 2

### Evasion 2 - Challenge

What is the MD5 hash of the backdoor executable?

### Evasion 2 - Solution

Unfortunately the file has been deleted, so we can't just hash there.
Luckily Linux provides us an easy facility to access the executable as if it were a file as it was loaded into memory via the `exe` virtual file.

{% highlight bash %}
md5sum /proc/12/exe
{% endhighlight %}

**Flag:** `c5a3bda03f01fe83815862a42f6e59da`

## Evasion 3

### Evasion 3 - Challenge

Which port is the backdoor listening on?

### Evasion 3 - Solution

Earlier we saw the backdoor had a file descriptor held open in `/proc/12/fd`: `3 -> 'socket:[5364808]'`.

We can cross-reference this to the files in `/proc/12/net`, in this case `tcp`.

``` text
root@df521907feb0:/proc/12/net# cat tcp
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode                                                     
   0: 00000000:0DA4 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 5364808 1 ffff8882037bb800 100 0 0 10 0
```

You can match up the number identified in the `fd` file link and the `inode` field of the `/proc/net/12/tcp` file.

We can see the socket is listening on `00000000:0DA4` which corresponds to `0.0.0.0:3492`.

For the conversion I like throwing the value into Python.

``` text
➜  ~ python3
Python 3.6.9 (default, Apr 18 2020, 01:56:04) 
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x0DA4
3492
>>>
```

**Flag:** `3492`
