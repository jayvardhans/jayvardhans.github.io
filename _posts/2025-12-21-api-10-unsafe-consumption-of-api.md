---
title: "API-10: Unsafe consumption of APIs"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-12-21-api-10-unsafe-consumption-of-api
tags:
- API Assessment
- API Pentesting
- API
- BOLA
- Broken Authentication
- Broken Object Property Level Authorization
- Security Misconfiguration
- OWASP API TOP 10
- Server-Side Request Forgery
- Unsafe consumption of APIs
- SSRF
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

## What is Unsafe consumption of APIs?

**Unsafe Consumption of APIs** occurs when an application **trusts and consumes data from third-party, partner, or internal APIs without properly validating the responses or enforcing adequate security controls**. Attackers can exploit weaknesses in the APIs an application relies on, causing the application to process malicious or unexpected data.

Modern applications often integrate with external services for payments, authentication, shipping, weather, social login, AI, and other functionality. If these integrations are not securely implemented, the application may inherit vulnerabilities from the APIs it consumes.

## Common Example

1. Trusting External API Responses
2. Processing Malicious Data
3. Sending Sensitive Data to Untrusted APIs
4. Insecure Error Handling

## Attack Scenario

In the application, no third-party, partner, or external APIs were identified that were consumed by the application. All API interactions were limited to internally hosted services; therefore, this category could not be evaluated.

## Mitigation

- Validate all API responses before processing or storing the data.
- Treat third-party APIs as untrusted, even if they are from trusted partners.
- Enforce TLS certificate validation and never disable certificate verification in production.
- Use strong authentication (such as OAuth 2.0, mutual TLS, or signed requests) for API-to-API communication.
- Implement input validation and output encoding for all data received from external APIs.
