---
title: ARP poisoning? Not on my watch!
excerpt: >-
  This article will explain you how to stop ARP poisoning from happening
categories:
  - security
tags:
  - Arp
  - Security
---
## Turn off gratious ARP

A gratious ARP policy means that Arp tables can be changed instantly whenever someone sends an ARP frame.
Turning this off would mean that any potential hacker has a very small window to get in front of the router arp when the arp cache gets refreshed.

## Use Dynamic ARP inspection

Dynamic ARP inspection (DAI) is a security feature that rejects invalid and malicious ARP packets. The feature prevents a class of man-in-the-middle attacks, where an unfriendly station intercepts traffic for other stations by poisoning the ARP caches of its unsuspecting neighbors. The miscreant sends ARP requests or responses mapping another station’s IP address to its own MAC address.
DAI relies on DHCP snooping. DHCP snooping listens to DHCP message exchanges and builds a bindings database of valid tuples (MAC address, IP address, VLAN interface).
When DAI is enabled, the switch drops ARP packet if the sender MAC address and sender IP address do not match an entry in the DHCP snooping bindings database. However, it can be overcome through static mappings. Static mappings are useful when hosts configure static IP addresses, DHCP snooping cannot be run, or other switches in the network do not run dynamic ARP inspection. A static mapping associates an IP address to a MAC address on a VLAN.
