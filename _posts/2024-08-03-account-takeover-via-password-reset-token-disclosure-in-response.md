---
title: "Account Takeover via Password Reset Token Disclosure in Response"
categories:
- Mobile Application Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2024-08-03-account-takeover-via-password-reset-token-disclosure-in-response
tags:
- ATO
- Account Takeover
- Password Reset Flaw
- Password Reset Token in Response
- Mobile Application Security
- Mobile Pentesting
- Android Pentesting
- IOS Pentesting
- Flutter
- Bypass Flutter Android
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## The Finding

During a recent mobile application penetration test, I identified a critical vulnerability in the application's password reset workflow. The issue was, password reset API returned the reset token directly in its response along with the associated email address, allowing an attacker to obtain the token without requiring access to the victim's email account.

## The Setup

I configured Burp Suite to intercept traffic, submitted a valid email address to initiate the password reset process, and inspected the server's response. Instead of simply acknowledging that the request had been received, the API returned the password reset token directly in the response body in plaintext. The same reset token was also sent to the user's registered email address.

## The Exploit

**Step 1**: I request a password reset token for any email address, in my case I am using `victimsingh02@gmail.com`. API return password reset token on response body.

![resettokeninresponse](resettokeninresponse.png)

**Step 2**: Observe that same reset token is sent to associated email address.

![resettokeninemail](resettokeninemail.png)

**Step 3**: I copied the reset token from the API response and access the password reset URL in a mobile browser along with the token. The application successfully validated the token and redirected me to the password reset page, where I was able to set a new password for the associated account.

![validatetoken](validatetoken.png)

**Step 4**:  I entered a new password on the password reset page, and submitted the request. The server responded with an HTTP 200 OK status and the message "Update Password Success", confirming that the password had been successfully changed.

![passwordchangesuccessfull](passwordchangesuccessfull.png)

## The Danger

**Complete account takeover:** An attacker could reset passwords across any accounts whose email addresses he know or can enumerate.

**Privilege escalation.** If the application has higher privilege roles, attackers could target it and gain access to admin account rather than normal user.

## **The Remediation**

**Never return a password reset token in an API response:** The password reset token should sent only associated email address. Do not sent or reflect in response body.

**Token expire window:** Password reset token should be expire with in timeframe such as 20 or 30 min.

**Token Reuse**: Once the token is used, it shouldn’t be reuse again.
