Wallaby's: Nightmare (1.0.2)
============================

Source: https://www.vulnhub.com/entry/wallabys-nightmare-102,176/

* Name: HackDay: Albania
* Released: 22 Dec 2016
* Author: Waldo
* Series: Wallaby's

Recon, Recon, Recon
-------------------

This VM was absolutely not connected to my network -- I set it on its own VLAN. As should you if you run any of these Vulnhub exercises.
I started by running a ping scan across my local `/16` to see what was open.

```
nmap -sn 192.168.0.0/16
```

That came up with 2 results for the VM:

```
map scan report for 192.168.71.129
Host is up (0.00040s latency).
MAC Address: 00:0C:29:6B:EE:FA (VMware)
Nmap scan report for 192.168.71.130
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

Let's check out this web app, shall we?

