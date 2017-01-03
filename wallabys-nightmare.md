Wallaby's: Nightmare (1.0.2)
============================

Source: https://www.vulnhub.com/entry/wallabys-nightmare-102,176/

* Name: HackDay: Albania
* Released: 22 Dec 2016
* Author: Waldo
* Series: Wallaby's

Recon, Recon, Recon
-------------------

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



