---
title: "Notes on the Pluralsight Kali Course"
excerpt: "This article will contain my personal notes on the kali course over at pluralsight"
categories:
 - Penetration-Testing
tags:
 - pentest

---

<center> [Link to the video Series](https://app.pluralsight.com/library/courses/kali-linux-penetration-testing-ethical-hacking/table-of-contents) </center>

#### Checking for rootkits:

 you can check for rootkits on kali by using the ```chkrootkit``` command in terminal.

#### Step 1: Information Gathering:

 ![Image of Information gathering workflow](/assets/images/infoworkflow.png)



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
