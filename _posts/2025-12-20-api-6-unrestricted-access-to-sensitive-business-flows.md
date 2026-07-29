---
title: "API-6: Unrestricted Access to Sensitive Business Flows"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-12-20-api-6-unrestricted-access-to-sensitive-business-flows
tags:
- API Assessment
- API Pentesting
- API
- BOLA
- Broken Authentication
- Broken Object Property Level Authorization
- Unrestricted Resource Consumption
- Broken Function Level Authorization (BFLA)
- Unrestricted Access to Sensitive Business Flows
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

## What is Unrestricted Access to Sensitive Business Flows?

**Unrestricted Access to Sensitive Business Flows** is an API security vulnerability that occurs when an API **fails to protect critical business processes from automated abuse or excessive use**. While the API may correctly authenticate and authorize users, it does not implement controls to prevent attackers from abusing legitimate business functionality at scale.

Unlike technical vulnerabilities such as SQL Injection or Broken Authentication, this weakness targets the **business logic** of the application.

## Common Vulnerable Scenarios

1. Automated Purchase During Flash Sales
2. Unlimited Coupon Redemption
3. Unlimited Account Registration
4. Mass Appointment Booking
5. Bulk Ticket Booking

## Attack Scenario

To exploit this vulnerability, I used fund transfer endpoint. Log in to the application as victim user and check available amount. Amount **2000** is available.

![victimuserfund](victimuserfund.png)

Now, I transfer amount negative **-900** from **victimuser to attackeruser** account.

![transfernegativeamount](transfernegativeamount.png)

Amount is successful transfer, and can see available amount is **2900**. To double check, I invoke check_balance endpoint and view available amount in **victimuser**.

![negativeamounttransfered](negativeamounttransfered.png)

Fund transfer was completed successfully, and the account balance was updated from **2000 to 2900**. I then verified the balance of the **attackeruser** account.

![amountattackeruser](amountattackeruser.png)

Attackeruser amount is now **-900**.

## Mitigation

- Implement Business Rate Limits
- Deploy CAPTCHA
- Restrict the number or value of transactions that a user can perform within a defined time period.



