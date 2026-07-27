---
title: "Vuln Bank API Penetration Testing"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-11-28-vuln-bank-api-penetesting
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

## Introduction

In this series, we will explore the OWASP API Security Top 10 vulnerabilities through hands-on demonstrations. For each vulnerability, we will understand the underlying concept, identify the security flaw, exploit it in a controlled environment, and discuss effective remediation techniques.

To demonstrate these attacks, we will use Vuln-Bank, an intentionally vulnerable banking application developed by Al Amir Badmus (Commando-X). It provides an excellent environment for learning and practicing real-world API security testing techniques.

You can find the project on GitHub:
<a href="https://github.com/Commando-X/vuln-bank">Vuln-Bank on GitHub</a>

## Lab Setup

I deployed Vuln-Bank locally using Docker Compose. The application includes a Swagger/OpenAPI specification, which I imported into Postman to automatically generate a complete collection of the available API endpoints.

![apidoc](apidoc.png)

For this assessment, I followed the OWASP API Security Top 10 testing methodology to systematically evaluate the application's security posture.

## Tools Used

The following tools were used throughout the assessment:

**Postman**

Postman was used to organize, send, and manage API requests. After cloning the Vuln-Bank repository, I imported the provided Swagger/OpenAPI documentation into Postman, which generated a ready-to-use collection containing every exposed API endpoint. This served as a comprehensive map of the application's API surface.

**Burp Suite**

Burp Suite was configured as an intercepting proxy between Postman and the Vuln-Bank application. While Postman was responsible for sending requests, Burp Suite allowed me to intercept, inspect, and modify them before they reached the server. This made it possible to manipulate parameters such as account IDs, inject test payloads, replay requests, and analyze server responses for potential vulnerabilities.

With this testing pipeline in place—Postman for crafting requests, Burp Suite for intercepting and modifying traffic, and Vuln-Bank as the target application—I was ready to begin the API security assessment by identifying and exploiting vulnerabilities mapped to the OWASP API Security Top 10.
