---
title: "Account Takeover Using Password Reset Token"
categories:
- Misc
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2026-07-25-account-takeover-using-password-reset-token
tags:
- ATO
- Account Takeover Vulnerability
- Password Reset Vulnerability
- Ethical hacking
- Bug Bounty
- Application Security
---

## What is Account Takeover vulnerability? 

Account Takeover vulnerability occurs when an attacker successfully gain the access of victim’s account and perform all the task.

## Description

I was testing an application and noticed that the forgot password functionality is vulnerable. It allows an attacker to manually craft a password reset token, and use it to reset a victim’s account password, and take full control of the account.

## Attack Scenario

When I tried to reset the password using forgot password functionality. Application sent a password reset token if email id exist.
I sent a password reset token to my email id - `singh.jayvardhan02@gmail.com`.

![reset-token](reset-token.png)

Below is the password reset token which I received.

```
https://******.com/api/v1/reset-password/9ISGvv8gqa7z5eccc04942b9c440257a36fe28381632b70d973b4ac9

```

After some time, I sent another password reset token to same email ID, and received below token.

```
https://******.com/api/v1/reset-password/9ISGvv8gqa7z5eccc04942b9c440257a36fe28381632b70d973b4ac9

```

Noticed that both the password reset tokens are same. Analyzing this, I sent password reset token to my another account..

Password reset token for email id - `jayvardhansingh02@gmail.com`

```
https://***.com/api/v1/reset-password/9ISGvv8gqa7z5ecc2cc927bf84a80e646d5e782eeeea853ff61d7cf9f1e

```

After analyzing the password reset token, I observed that first 16 characters are identical in all the tokens, and rest is SHA1 hash value of the email ID.

If I break the tokens, first 16 character are same for both email id - ```9ISGvv8gqa7z5ecc```

SHA1 hash for `singh.jayvardhan02@gmail.com` - ```c04942b9c440257a36fe28381632b70d973b4ac9```

SHA1 hash for `jayvardhansingh02@gmail.com` - ```927bf84a80e646d5e782eeeea853ff61d7cf9f1e```

After that, I created another account (victim's account) using email id `sectest24@gmail.com`

Based on above observation, I crafted a password reset token manually for the victim’s account.

Password reset token for victim’s email id - `sectest24@gmail.com`

```
https://*****.com/api/v1/reset-password/9ISGvv8gqa7z5ecc2cc046541723cb440b5de0bfb9afa9649c56bcef

```

Using above crafted password reset link, I successfully able to change the victim's account password.

## Recommendation

Generate a one-time token linked to the user account. Create random token instead of the email-ID hash, and expire it immediately after the password is reset.
