---
title: "API-4: Unrestricted Resource Consumption"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-12-18-api-4-unrestricted-resource-consumption
tags:
- API Assessment
- API Pentesting
- API
- BOLA
- Broken Authentication
- Broken Object Property Level Authorization
- Unrestricted Resource Consumption
- BOPLA
- VulnBank
- API OWASP Top 10
- API Security
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## What is Unrestricted Resource Consumption?

**Unrestricted Resource Consumption** is an API security vulnerability that occurs when an **API fails to limit the consumption of system resources** such as CPU, memory, storage, bandwidth, database connections, or cloud services. Attackers can abuse these resources by sending excessive requests or large payloads, causing the application to slow down, crash, or generate excessive operational costs.

This vulnerability was previously known as **Lack of Resources & Rate Limiting** in older versions of the OWASP API Top 10, but in the latest version it has been expanded to include **all forms of uncontrolled resource usage**, not just request rates.

## Common Vulnerable Scenarios

1. No Rate Limiting
2. Large File Upload
3. Unlimited Pagination
4. Expensive Search Queries

## Attack Scenario

To exploit this vulnerability, I used the Forgot Password endpoint. When I invoked the endpoint, a reset PIN was sent to the email address associated with the account.

![pinsent](pinsent.png)

Now try to reset the password. I put OTP ‘000’ and sent the request, received the response “Invalid reset PIN”. 

![pininvalid](pininvalid.png)

I sent the request into the burp intruder and configure it.

![pinbruteconfig](pinbruteconfig.png)

Configure the brute force attack and launch it. I successfully brute force the PIN after 925 requests.

![pinbrutesuccess](pinbrutesuccess.png)

I received the response "Password has been reset successfully".

## Mitigation

- Implement rate limiting
- Restrict request payload size.
- Limit file upload size.
- Enforce pagination limits.
- Configure request and query timeouts.
