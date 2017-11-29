---
title: Notes on the Pluralsight Kali Course
excerpt: >-
  This article will contain my personal notes on the kali course over at
  pluralsight
categories:
  - Penetration-Testing
tags:
  - pentest
---

[Link to the video Series](https://app.pluralsight.com/library/courses/kali-linux-penetration-testing-ethical-hacking/table-of-contents)

# Checking for rootkits:

you can check for rootkits on kali by using the `chkrootkit` command in terminal.

# Step 1: Information Gathering:

![workflow](https://jeanfrancoismaes.github.io/work-adventures/assets/images/infoworkflow.png)

- Always check the website first. You can learn a lot from there already. Stuff like email, sometimes even management information, check the job listings to see what kind of technology they use.

- Kali tools for information gathering include (but are not limited to):

  - The Harvester
  - DNSRecon
  - Dmitry
  - Recon-ng

- Prepare a dedicated workspace for the documentation, you can use the built in application **keepnote** for this purpose, a nice skeleton would be

  - Information Gathering
  - External Pentest
  - Internal Pentest

- Tools like Google Dorks together with the vulnrabilityDB can sometimes be used to extract some sensitive data without much efford.

## Checking for a Web Application Firewall (WAF)

There is a nice tool in Kali that will tell you if your webapplication is behind a Web Application Firewall, It's called `wafw00f` using this tool is as simple as just typing `wafw00f http://example.com` in your kali terminal

If the application is behind a WAF the output would look like this:

{% capture fig_img %} ![wafwoofscreenshot]({{ "/assets/images/wafwoofpositive.png" | absolute_url }}) {% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>this website is behind a WAF</figcaption></figure>

## Check for loadbalancers

It's nice to know if your target does loadbalancing, and if so, what kind of loadbalncing it's doing. To check this out we can use the tool `lbd` If the application uses loadbalancing you would get a result similar to this one: (open the image in new tab for full scale size)

{% capture fig_img %} ![wafwoofscreenshot]({{ "/assets/images/lbdpositive.png" | absolute_url }}) {% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>this website is using LoadBalancing</figcaption></figure>

## WebCrawling

In order to understand the web structure better (and in order to find hidden pages (admin pages, config pages,...) **if you have the pro version**) we can use Burpsuite as an intercepting proxy The problem is that burpsuite breaks https traffic, in order to fix that you need to put burp's certificate as trusted root certificate in your browser, you can follow this link if you use firefox as your pen testing browser: [how to install burp cert on firefox](https://support.portswigger.net/customer/portal/articles/1783087-Installing_Installing%20CA%20Certificate%20-%20FF.html)

The spider tab in burpsuite can crawl the webpage you are on, if there are forms on the webpage burp will ask you for credentials, this can be annoying when there are a lot of forms in the webapplication. you can change this behavior in the burpsuite options and change it to a basic SQL injection:

```
username: admin' or 1=1 --
password: blank
```

{% capture fig_img %} ![wafwoofscreenshot]({{ "/assets/images/burplogin.png" | absolute_url }}) {% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>standard SQL Injection setting in burp</figcaption></figure>

## Website ripping

HTTrack can be downloaded in kali (not installed by default) to shamelessly clone websites

## Certification Tests

If the website uses HTTPS (which it should, its 2017 people..) you can use SSLScan to see information about the certificate and see if heartbleed can still be used.

## CMSscan

There are tools in kali to pen test CMS, in case of wordpress you can use the `WPscan` tool to brute force user logins.

## Vulnerability Scanner

Vulnerability scanners are automated tools to check for well known vulnerabilities, keep in mind however that vulnerabilty scanners are usually quite "loud" which means that they will most likely get detected by the target.

Burp has a vulnerability scanner **pro version only** and can be found under the "scanner" tab in the burpsuite tool.

# Step 2: External Pen Tests

In this step we will use the information gathered in step 1 to start looking for "a way in". The most popular tool to use is nmap to check for active hosts in the network and to port scan any open ports.

for more information on nmap, check the nmap man page.

# Step 3: Website Pen testing

## Session Tokens

Session tokens should be randomized, and burpsuite can check if this is the case. surf to a website that uses sessions (such as mutilidae, a vulnerable web application which can be accessed from the exploitable vm on sourceforge) and intercept traffic with burp, forward the request and in the response you should get a cookie, now you can send this response to the burp sequencer with right click - send to sequencer, and you can generate new tokens, you can then analize the generated tokens with the analyze now button

## SQL Injection (Sqlmap)

Sqlmap is a python tool that automates SQL attacks.

you can use mutilidae for this attack as a test host, but there is a small "bug" you'd have to fix first. ssh into the metasploitable machine and change the $dbname in the config file `/var/www/mutillidae/config` to owasp10 go to the user-info.php page on the site and capture the login request with burp save the content of the request to a file and call it for this example "request.txt".

open terminal, and issue the following command: `sqlmap -r /root/Burp\ Suite/request.txt --dbs` to determine the available db's.
you should also see that username is the weaklink so you can use `-p username` in sqlmap now too.

`sqlmap -r /root/Burp\ Suite/request.txt -D owasp10 --tables` shows the available tables in the owasp10 DB.

`sqlmap -r /root/Burp\ Suite/request.txt -D owasp10 -T accounts --dump` dumps the accounts table

## Maintaining access

To insure that access can be maintained, you can use webshells, Kali has some good tools for this. In this example we use Weevely. Weevely has a lot of functions, which include but are not limited to:

- File browsing
- File Transfer
- Auditing
- Compromising SQL Servers
- Reverse TCP shells
- command execution

`weevely generate hacker #creates a backdoor file with password hacker`

now the file needs to get uploaded onto the webserver, and you need to know the url to access the file. in our testlab we can use the upload page on the DVWA on metasploitable when security settings are set on low.

When all of this is done you can use `weevely <url to weevelyfile> <password>`
After executing this properly you are now in the webserver with a remote shell. And then you can do whatever you'd like... :)

## Dos attacks

LOIC is a nice tool for DDOS attacks but requires you to have ```wine``` installed since it's a .exe
I found that slowhttptest is a nice tool as well, and is installed using apt.

# Step 4: internal pen Testing

For internal pen testing you can use a Vulnerability scanner such as openvas and nmap to see open ports, live hosts, ...
be careful when using nmap because it can slow down the network considerably, you can use ``` -T ``` parameter, T1 is the slowest, T5 is the fastest.

If you haven't used openvas before, you need to execute the setupscripts first, this could take some time.

# Step 5: Network Sniffing
Wireshark can capture all packets transmitted, all you need is a NIC in promisculous mode, this will allow the NIC to see all packets, not just the packets addressed to them.

Wireshark can be used as a monitor tool but this will require you to have profound knowledge of the network:
 * IP ranges
 * Communication Equipment
 * Security Devices
 * Applications and their portnumbers

 {% capture fig_img %} ![weirdtraffic]({{ "/assets/images/suspicious.png" | absolute_url }}) {% endcapture %}

 <figure>
   {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
   <figcaption>it might be usefull to take a look in these packets...</figcaption></figure>


## Sniffing methods

{% capture fig_img %} ![sniffing]({{ "/assets/images/sniffing.png" | absolute_url }}) {% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>The most common sniffing methods</figcaption></figure>


### Detecting Man-In-The-Middle (MITM)

As seen on the Image, one of the commonly used methods of sniffing the network is a MITM attack, this kind of attack has lots of permutations.
