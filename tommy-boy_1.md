Tommy Boy: 1
============

Source: https://www.vulnhub.com/entry/tommy-boy-1%2C157/

* Name: Tommy Boy: 1
* Date release: 27 Jul 2016
* Author: Brian Johnson
* Series: Tommy Boy
* Web page: http://7ms.us/tommyboy

Of course I had to pick this one, I loved this movie as a kid. After some fiddling on my testing VM, I'll be ready to start recon.

**Note:** 
Some parts of the CTF took a bit to load, as it was trying to reach external assets while still connected to my 
local network. Just a heads-up if you're experiencing the same thing.


Brothers Don't Complete the Handshake, Brothers Gotta Hug
---------------------------------------------------------

`nmap` time. Running it locally only, so I check the local network, which took less than 2 minutes:

`nmap -sV -A 192.168.0.0/16 -vv`

From there, I find my target:

```
Nmap scan report for 192.168.xx.xx
Host is up, received arp-response (0.00037s latency).
Scanned at 2017-01-20 11:57:16 CST for 87s

[insert brevity throughout]

PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 64 OpenSSH 7.2p2 Ubuntu 4ubuntu1 (Ubuntu Linux; protocol 2.0)

80/tcp   open  http    syn-ack ttl 64 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
| http-robots.txt: 4 disallowed entries 
| /6packsofb...soda /lukeiamyourfather 
|_/lookalivelowbridge /flag-numero-uno.txt
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to Callahan Auto
8008/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: KEEP OUT

111/tcp open  rpcbind syn-ack ttl 64 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          40605/udp  status
|_  100024  1          49865/tcp  status
```

There's a `robots.txt` file intentionally blocking a `.txt` file for a flag. Time to check it out by going to http://192.168.xx.xx/flag-numero-uno.txt.

```
This is the first of five flags in the Callhan Auto server.  You'll need them all to unlock
the final treasure and fully consider the VM pwned!

Flag data: B34rcl4ws
```

Time to check the others, too!

http://192.168.xx.xx/lukeiamyourfather/ has only an image of Tommy Callahan flipping me off with the skeleton hand. Classic. Also noted that indexing of the folder is allowed. Anything in the image, though?

`identify -verbose tmf.jpg|grep -i exif`

Hm, no exif data. And now Apache isn't responding? So, I just rebooted the VM. You're not my supervisor.

After some review, all the subfolder images are normal, with nothing I could find hidden. I'm probably just focusing too hard on that. Good movie memories, though!

Not Really Redundant
--------------------
Looking back at earlier `nmap` results, it shows Apache running on two ports. Why? What's on `:8008`?

http://192.168.xx.xx:8008

> This is only for Nick's super secret stuff. If you don't know where to go from
> here, you're not sup3rl33t enough.
> 
> Leave now!
> 
> Only me and Steve Jobs are allowed to look at this stuff.
> 
> Lol
> 
> -Nick

Nothing in source here.

But what about on that first page I didn't check?

```
<html>
<title>Welcome to Callahan Auto</title>
<body>
<H1><center>Welcome to Callahan Auto!</center></H1>
<font color="FF3339"><H2>SYSTEM ERROR!</H2></font>
If your'e reading this, the Callahan Auto customer ordering system is down.  Please restore the backup copy immediately.
<p>
See Nick in IT for assistance.
</html>
<!--Comment from Nick: backup copy is in Big Tom's home folder-->
<!--Comment from Richard: can you give me access too? Big Tom's the only one w/password-->
<!--Comment from Nick: Yeah yeah, my processor can only handle one command at a time-->
<!--Comment from Richard: please, I'll ask nicely-->
<!--Comment from Nick: I will set you up with admin access *if* you tell Tom to stop storing important information in the
company blog-->
<!--Comment from Richard: Deal.  Where's the blog again?-->
<!--Comment from Nick: Seriously? You losers are hopeless. We hid it in a folder named after the place you noticed after you
and Tom Jr. had your big fight. You know, where you cracked him over the head with a board. It's here if you don't remember:
https://www.youtube.com/watch?v=VUxOd4CszJ8--> 
<!--Comment from Richard: Ah! How could I forget?  Thanks-->
```
Two clues here:

* storing important information in the company blog
* https://www.youtube.com/watch?v=VUxOd4CszJ8

Well, the 2nd point brings us to the epic battle in front of... Prehistoric Forest. I assumed the URL was
http://192.168.xx.xx/prehistoricforest, and that worked! 

There's a lone comment at the top post:

```
richard says:	
July 7, 2016 at 6:04 pm

Hey numbnuts, look at the /richard folder on this server. I’m sure that picture will jog your memory.

Since you have a small brain: see up top in the address bar thingy? Erase “/prehistoricforest” and put “/richard” 
there instead.
```

"Richard, what'd'ja do???"

Here I see http://192.168.xx.xx/richard/shockedrichard.jpg has classic Richard about to hit a deer. Out of futility, 
I tried to check exif data once more:

`~# identify -verbose Downloads/shockedrichard.jpg|grep -i exif`
```
    exif:Copyright: Copyright .. 1995 Paramount Pictures Corporation. Credit: .. 1995 Paramount Pictures / Courtesy: Pyxurz.
    exif:ExifImageLength: 1029
    exif:ExifImageWidth: 1600
    exif:ExifOffset: 164
    exif:ExifVersion: 48, 50, 50, 48
    exif:Software: Google
    exif:UserComment: 65, 83, 67, 73, 73, 0, 0, 0, 99, 101, 49, 53, 52, 98, 53, 97, 56, 101, 53, 57, 99, 56, 
    57, 55, 51, 50, 98, 99, 50, 53, 100, 54, 97, 50, 101, 54, 98, 57, 48, 98
    Profile-exif: 264 bytes
```
Hm, that comment isn't too easy to read. Let me try `exiftool`:

`~# exiftool Downloads/shockedrichard.jpg|grep -i comment`
```
User Comment                    : ce154b5a8e59c89732bc25d6a2e6b90b
```

That's better. Time to crack this hash!

Did I Catch a "Niner" in There?
-------------------------------

`~# john --wordlist=/usr/share/wordlists/rockyou.txt --rules richard.txt --format=Raw-MD5`
```
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 AVX 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
spanky           (?)
1g 0:00:00:00 DONE (2017-01-23 14:19) 25.00g/s 34500p/s 34500c/s 34500C/s lacoste..spanky
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

That was fast! Looks like it's `spanky`. Oh Richard...

Not going to put all the post contents for brevity, but here are relevant portions.

**Backups:**
```
But, thanks to *me* there’s a backup called callahanbak.bak that you can just rename to index.html and everything will 
be good again.

    IMPORTANT: You have to do this under Big Tom’s account via SSH to perform this restore. 
```
**FTP?:**
```
Basically I couldn’t get it running on the standard port, so I put it on a port that most scanners would get exhausted 
looking for.  And to make matters more fun, the server seems to go online at the top of the hour for 15 minutes, then 
down for 15 minutes, then up again, then down again.
```
Oh good, more "security through obscurity".

**Nick Burns, your company's computer guy:**
```
I just reset my account (“nickburns” in case you’re dumb and can’t remember) to a very, VERY easy to guess password.
```

FTP = Freshener: Trees of Pine
------------------------------
Trying too hard on these section titles. Time to find that FTP instance!

`~# nmap -vv 192.168.xx.xx -p1-65535 --open`

```
Reason: 65531 resets
PORT      STATE SERVICE REASON
22/tcp    open  ssh     syn-ack ttl 64
80/tcp    open  http    syn-ack ttl 64
8008/tcp  open  http    syn-ack ttl 64
65534/tcp open  unknown syn-ack ttl 64
```

(Btw, unsure if FTP is up? Just check real quick with `telnet`!)
```
~# telnet 192.168.xx.xx 65534
Trying 192.168.xx.xx...
telnet: Unable to connect to remote host: Connection refused
```
^ *NARP*
```
~# telnet 192.168.xx.xx 65534
Trying 192.168.xx.xx...
Connected to 192.168.xx.xx.
Escape character is '^]'.
220 Callahan_FTP_Server 1.3.5
```
^ *YARP*

Normally it'd be a job for `hydra`, but since the FTP server cycles twice per hour (up for 15 minutes, and down for 15), that sounds very very annoying.

`~# hydra -l nickburns -P /usr/share/wordlists/rockyou.txt ftp://192.168.xx.xx:65534`

Easy to guess, huh? I tried a few, and, it ends up being `nickburns`. That sounds about right. 

```
230 User nickburns logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful
150 Opening ASCII mode data connection for file list
-rw-rw-r--   1 nickburns nickburns      977 Jul 15  2016 readme.txt
ftp> get readme.txt
local: readme.txt remote: readme.txt
200 PORT command successful
150 Opening BINARY mode data connection for readme.txt (977 bytes)
226 Transfer complete
977 bytes received in 0.00 secs (422.1688 kB/s)
ftp> quit
221 Goodbye.

[pause]

~# cat readme.txt 
To my replacement:

If you're reading this, you have the unfortunate job of taking over IT responsibilities
from me here at Callahan Auto.  HAHAHAHAHAAH! SUCKER!  This is the worst job ever!  You'll be
surrounded by stupid monkeys all day who can barely hit Ctrl+P and wouldn't know a fax machine
from a flame thrower!

Anyway I'm not completely without mercy.  There's a subfolder called "NickIzL33t" on this server
somewhere. I used it as my personal dropbox on the company's dime for years.  Heh. LOL.
I cleaned it out (no naughty pix for you!) but if you need a place to dump stuff that you want
to look at on your phone later, consider that folder my gift to you.

Oh by the way, Big Tom's a moron and always forgets his passwords and so I made an encrypted
.zip of his passwords and put them in the "NickIzL33t" folder as well.  But guess what?
He always forgets THAT password as well.  Luckily I'm a nice guy and left him a hint sheet.

Good luck, schmuck!

LOL.

-Nick
```

Remembering the `:8008` landing page from earlier (haha, boob, I get jokes), I try to append that l33tsp34k folder to the address, and I get this:

http://192.168.xx.xx:8008/NickIzL33t

> Nick's sup3r s3cr3t dr0pb0x - only me and Steve Jobs can see this content
> Lol



