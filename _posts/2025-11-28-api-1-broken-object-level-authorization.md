---
title: "API-1: Broken Object Level Authorization"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2024-03-28-vuln-bank-api-penetesting
tags:
- API Assessment
- API Pentesting
- API
- BOLA
- Broken Object Level Authorization
- VulnBank
- API OWASP Top 10
- API Security
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## What is Broken Object Level Authorization?

**Broken Object Level Authorization (BOLA)** is a security vulnerability that occurs when an application **fails to verify whether a user is authorized to access a specific object (resource)**. As a result, an attacker can manipulate object identifiers (such as user IDs, order IDs, or document IDs) to access data belonging to other users.

## Attack Scenario

Using the `/register ` endpoint. I have created two account with username `“victimuser”` and `“attackeruser”`.

Creating account for `attackeruser`

![attackeruser](attackeruser.png)

Attacker account details

```
username - attackeruser
password - password123
Account -  0473947940
Userid - 811
```

Creating account for `victimuser`

![victimuser](victimuser.png)

Victim account details

```
username - victimuser
password - pass123
Account - 0812131673
Userid- 809
```
