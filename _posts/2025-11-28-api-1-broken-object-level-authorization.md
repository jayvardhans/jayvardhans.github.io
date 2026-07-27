---
title: "API-1: Broken Object Level Authorization"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-11-28-api-1-broken-object-level-authorization
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

Attacker account information

```
username - attackeruser
password - password123
Account -  0473947940
Userid - 811
```

Creating account for `victimuser`

![victimuser](victimuser.png)

Victim account information

```
username - victimuser
password - pass123
Account - 0812131673
Userid- 809
```

First I login as an attacker to hit endpoint `/login` and do some transaction. I transfer amount `100` from `attackeruser` to `victimuser`. I have also noticed that JWT token is also refelected in response body.

![loginattacker](login attacker.png)

Transfer amount `100` from `attackeruser` to `victimuser`.

![transfermoney](transfermoney.png)

Now check the transaction history of `attackeruser`.

![transationhistoryattacker](transationhistoryattacker.png)

In the response we can see that amount 100 is successfully transfer to `victimuser` account.

In the Transaction History API endpoint, I replaced the authenticated user's account number with the victim's account number in GET URL. After sending the modified request, the API returned the victim's transaction history, even though I was logged in as the attacker user.

![transationhistoryvictim](transationhistoryvictim.png)

This demonstrates a Broken Object Level Authorization (BOLA) vulnerability. The application failed to verify whether the authenticated user was authorized to access the requested account, allowing an attacker to retrieve another user's sensitive financial information simply by modifying the account identifier in the request.

## Mitagation

- Enforce Server-Side Authorization Checks
- Implement Role-Based or Attribute-Based Access Control
- Do Not Trust User-Supplied Object Identifiers
