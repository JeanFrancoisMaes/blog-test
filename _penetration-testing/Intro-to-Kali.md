---
title: "Notes on the Pluralsight Kali Course"
excerpt: "This article will contain my personal notes on the kali course over at pluralsight"
categories:
 - Penetration-Testing
tags:
 - pentest

---

[Link to the video Series](https://app.pluralsight.com/library/courses/kali-linux-penetration-testing-ethical-hacking/table-of-contents)

### Checking for rootkits:

 you can check for rootkits on kali by using the ```chkrootkit``` command in terminal.

### Step 1: Information Gathering:

![workflow](https://jeanfrancoismaes.github.io/work-adventures/assets/images/infoworkflow.png)


* Always check the website first. You can learn a lot from there already. Stuff like email, sometimes even management information, check the job listings to see what kind of technology they use.

* Kali tools for information gathering include (but are not limited to):
	* The Harvester
	*  DNSRecon
	*  Dmitry
	*  Recon-ng

* Prepare a dedicated workspace for the documentation, you can use the built in application **keepnote** for this purpose, a nice skeleton would be
	* 	Information Gathering
	* 	External Pentest
	* 	Internal Pentest

* Tools like Google Dorks together with the vulnrabilityDB can sometimes be used to extract some sensitive data without much efford.

#### Checking for a Web Application Firewall (WAF)

There is a nice tool in Kali that will tell you if your webapplication is behind a Web Application Firewall, It's called ```wafw00f``` using this tool is as simple as just typing ```wafw00f http://example.com ``` in your kali terminal

If the application is behind a WAF the output would look like this:

{% capture fig_img %}
![wafwoofscreenshot]({{ "/assets/images/wafwoofpositive.png" | absolute_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>this website is behind a WAF</figcaption>
</figure>

#### Check for loadbalancers

It's nice to know if your target does loadbalancing, and if so, what kind of loadbalncing it's doing. To check this out we can use the tool ```lbd ```
If the application uses loadbalancing you would get a result similar to this one: (open the image in new tab for full scale size)

{% capture fig_img %}
![wafwoofscreenshot]({{ "/assets/images/lbdpositive.png" | absolute_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>this website is using LoadBalancing</figcaption>
</figure>

#### WebCrawling
In order to understand the web structure better and in order to find hidden pages (admin pages, config pages,...) we can use Burpsuite as an intercepting proxy
The problem is that burpsuite breaks https traffic, in order to fix that you need to put burp's certificate as trusted root certificate in your browser, you can follow this link if you use firefox as your pen testing browser: [how to install burp cert on firefox](https://support.portswigger.net/customer/portal/articles/1783087-Installing_Installing%20CA%20Certificate%20-%20FF.html)

The spider tab in burpsuite can crawl the webpage you are on, if there are forms on the webpage burp will ask you for credentials, this can be annoying when there are a lot of forms in the webapplication. you can change this behavior in the burpsuite options and change it to a basic SQL injection:
```
username: admin' or 1=1 --
password: blank
```

{% capture fig_img %}
![wafwoofscreenshot]({{ "/assets/images/burplogin.png" | absolute_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>standard SQL Injection settign in burp</figcaption>
</figure>

#### Website ripping
HTTrack can be downloaded in kali (not installed by default) to shamelessly clone websites

#### Certification Tests
If the website uses HTTPS (which it should, its 2017 people..) you can use SSLScan to see information about the certificate and see if heartbleed can still be used.

#### CMSscan
There are tools in kali to pen test CMS, in case of wordpress you can use the ```WPscan``` tool to brute force user logins.
