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
