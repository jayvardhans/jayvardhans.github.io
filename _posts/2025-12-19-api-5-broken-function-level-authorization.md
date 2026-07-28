---
title: "API-5: Broken Function Level Authorization (BFLA)"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-12-19-api-5-broken-function-level-authorization
tags:
- API Assessment
- API Pentesting
- API
- BOLA
- Broken Authentication
- Broken Object Property Level Authorization
- Unrestricted Resource Consumption
- Broken Function Level Authorization (BFLA)
- BFLA
- BOPLA
- VulnBank
- API OWASP Top 10
- API Security
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## What is Broken Function Level Authorization?

**Broken Function Level Authorization (BFLA)** is an API security vulnerability that occurs when an **API fails to properly enforce authorization checks for specific functions or actions**. As a result, a user with lower privileges (or even an unauthenticated user) can access functionality that should only be available to users with higher privileges, such as administrators or privileged roles.

Unlike **Broken Object Level Authorization (BOLA)**, which controls what data a user can access, BFLA controls **what actions or functions a user is allowed to perform**.

## Common Vulnerabilities

1. Privilege Escalation
2. HTTP Method Manipulation
3. Role Manipulation
4. Insecure Password Reset Functions

## Attack Scenario

To exploit this vulnerability, I log in as victimuser account. Capture the JWT and decode it using `jwt.io`.

![loginasvictimuser](loginasvictimuser.png)

Analyze JWT and modify payload `is_admin: true`, copy new token.

![modifyjwt](modifyjwt.png)

Invoked the **Delete User** endpoint using new generated JWT. This functionality should be restricted to admin; only users with admin privileges can authorized to delete user accounts.

![deleteuser](deleteuser.png)

We successfully delete the account with victimuser.

Now, invoke `Create Admin` endpoint, only admin has privilege to create admin. I use new generated JWT.

![createadmin](createadmin.png)

I successfully create the admin using `victimuser` account.

## Mitigation

- Enforce Server-Side Authorization
- Implement Role-Based Access Control (RBAC)
- Validate Authorization for Every Endpoint
