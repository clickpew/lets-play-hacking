Tommy Boy: 1
============

Source: https://www.vulnhub.com/entry/tommy-boy-1%2C157/

* Name: Tommy Boy: 1
* Date release: 27 Jul 2016
* Author: Brian Johnson
* Series: Tommy Boy
* Web page: http://7ms.us/tommyboy

Of course I had to pick this one, I loved this movie as a kid. After some fiddling on my testing VM, I'll be ready to start recon.


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

There's a `robots.txt` file intentionally blocking us from a `.txt` file for a flag. Time to check it out by going to http://192.168.xx.xx/flag-numero-uno.txt.

```
This is the first of five flags in the Callhan Auto server.  You'll need them all to unlock
the final treasure and fully consider the VM pwned!

Flag data: B34rcl4ws
```
