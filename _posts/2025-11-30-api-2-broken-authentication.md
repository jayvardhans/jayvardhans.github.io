---
title: "API-2: Broken Authentication"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-11-30-api-2-broken-authentication
tags:
- API Assessment
- API Pentesting
- API
- BOLA
- Broken Authentication
- VulnBank
- API OWASP Top 10
- API Security
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## What is Broken Authentication?

**Broken Authentication** is a security vulnerability that occurs when an API fails to **properly verify the identity of users or manage authentication securely**. This allows attackers to impersonate legitimate users, hijack accounts, or gain unauthorized access to sensitive data and functionality.

Unlike **Broken Object Level Authorization (BOLA)**, which is about **what a user can access, Broken Authentication is about whether the application correctly verifies who the user is**.

## Scenario - 1

In this attack, I checked SQL injection to bypass authentication in the login endpoint `/login`. We successfully bypass authentication mechanism and successfully able to login another user account.

![sqlauth](sqlauth.png)

## Scenario - 2

In this attack we invoke the forgot password endpoint and check how API respond? Notice that a reset PIN is sent to the respected email id.

![v3emailsent](v3emailsent.png)

And another thing I notice that, in this endpoint API v3 is being used. What if I downgrade this version to v1.

![v1emailsent](v1emailsent.png)

Notice that after modifed the API version from v3 to v1, debug information is display in the response with the password reset pin of victimuser.

We can use this pin to reset `victimuser` password and take full control of `victimuser` account.

## Mitgation

- Use parameterized queries or prepared statements.
- Validate and sanitize all user input.
- Avoid dynamically constructing SQL queries.
- Disable and remove deprecated API versions.
- Never return password reset tokens or PINs in API responses.
- Disable debug mode in production environments.
