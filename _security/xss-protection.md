---
title: How to protect from cross site scripting
excerpt: >-
  Cross Site scripting can be nasty, how do you protect your website against it?
categories:
  - security
tags:
  - Security
  - XSS
---
## What is XSS?
Cross site scripting is the act of injecting malicious code into a website, which is not coded to catch code blocks as malicious, hence executing the code.
A simple example would be a `<script>alert("haha you got hacked!")</script>` comment on a messageboard. If the message board has no way of treating this as just text, the website will treat it like legit code, and everytime a user browses that page, he would recieve a pop up saying haha you got hacked.

## How to fix this issue?
There are a few ways to protect your website against these kinds of attacks. OWASP has some ground rules on how to code your application to be as secure as possible against these kinds of attacks. More about these rules in the section below, first it's important to understand there are 2 approaches to tackle this problem:

**blacklisting:** This is the hardest one to implement, you filter out all illegal characters and don't allow them. The reason why this is the hardest one to implement is because you'd have to include ALL the problematic characters, if you forget one, it's game over.

**whitelisting:** This approach starts with a deny all policy, and only allowing what you deem as valid content.

Basically the gist of it is that you'll have to escape your code correctly, with special consideration to the following code blocks:

{% capture fig_img %} ![rule1]({{ "/assets/images/rule1.png" | absolute_url }}) {% endcapture %}

{% capture fig_img %} ![rule2]({{ "/assets/images/rule2.png" | absolute_url }}) {% endcapture %}

{% capture fig_img %} ![rule3]({{ "/assets/images/rule3.png" | absolute_url }}) {% endcapture %}

{% capture fig_img %} ![rule4]({{ "/assets/images/rule4.png" | absolute_url }}) {% endcapture %}

More information can be found on the owasp website: [here](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet)
