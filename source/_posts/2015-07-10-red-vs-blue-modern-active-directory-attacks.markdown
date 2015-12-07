---
layout: post
title: "Red vs. Blue: Modern Active Directory Attacks"
date: 2015-07-10 09:08:51 +0900
comments: true
categories: 
---

From shakacon.org

Name: Sean Metcalf

Synopsis: While Kerberos "Golden Tickets" and "Silver Tickets" received a lot of press in the second half of 2014, there hasn't been much detail provided on how exactly they work, why they are successful, and how to mitigate them (other than: "don't get pwned"). Golden Tickets are the ultimate method for persistent, forever AD admin rights to a network since they are valid Kerberos tickets and can't be detected, right?

This talk covers the latest AD attack vectors and describes how to detect Golden Ticket usage. Provided are key indicators that can detect Kerberos attacks on your network, including Golden tickets, Silver tickets & MS14-068 exploitation, as well as methods to identify, mitigate, and prevent common Active Directory attack vectors. When forged Kerberos tickets are used in AD, there are some interesting artifacts that can be identified. Yes, despite what you may have read on the internet, there are ways to detect Golden & Silver Ticket usage!

Some of the topics covered:

* How attackers go fro mzero to (Domain) Admin
* MS14-068: the vulnerability, the exploit, and the danger
* "SPN Scanning" with PowerShell to identify potential targets without network scans (SQL, Exchange, FIM, webservers, etc)
* Exploiting weak service account passwords as a regular AD user
* Mimikatz, the attacker's multi-tool
* Using Silver Tickets for stealthy persistence that won't be detected (until now)
* Identifying forged Kerberos tickets (Golden & Silver Tickets) on your network
* Detecting offensive PowerShell tools like Invoke-Mimikatz
* Active Directory attack mitigation
