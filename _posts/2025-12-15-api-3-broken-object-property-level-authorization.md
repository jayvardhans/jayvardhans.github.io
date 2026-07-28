---
title: "API-3: Broken Object Property Level Authorization"
categories:
- API Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-12-15-api-3-broken-object-property-level-authorization
tags:
- API Assessment
- API Pentesting
- API
- BOLA
- Broken Authentication
- Broken Object Property Level Authorization
- BOPLA
- VulnBank
- API OWASP Top 10
- API Security
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## What is Broken Object Property Level Authorization (BOPLA)?

**Broken Object Property Level Authorization (BOPLA)** occurs when an API fails to enforce authorization at the **individual object property (field) level, allowing users to read or modify sensitive properties** they should not have access to.

Unlike **Broken Object Level Authorization (BOLA)**, where an attacker gains access to an entire object belonging to another user, BOPLA occurs when the attacker can access or manipulate **specific properties within an object** that should be restricted.

This issue consists of two independent security vulnerabilities

  1- Excessive Data Exposure
  2- Mass Assignment

## Excessive Data Exposure

Excessive Data Exposure is an API security vulnerability where an API **returns more data than is necessary** for a given request. Instead of filtering **sensitive information on the server, the API sends the entire object to the client**, relying on the client application to hide or ignore unnecessary fields. An attacker can inspect the raw API response to access sensitive information that should not be exposed.

During the user registration process, the API response exposes excessive sensitive information, including the username, password, account number, account balance, and other confidential data, which should not be returned to the client.

![Excessivedataexposure](Excessivedataexposure.png)

## Mass Assignment

Mass Assignment is an API vulnerability that occurs when an application automatically **binds user-supplied input to internal object properties without properly restricting which fields can be modified**. This allows attackers to update sensitive attributes that should not be accessible, such as user roles, account status, permissions, or administrative flags.

During the registration process, I observed that the application exposes the user role through the `is_admin` parameter in response, which is set to false by default. To test for a Mass Assignment vulnerability,  I registered a new user account  with the additional parameter `is_admin: true` included in the registration request.

![massassignment](massassignment.png)

The application successfully processed the modified registration request, allowing the creation of a new user account with administrative privileges by accepting the manipulated `is_admin: true` parameter. 

This confirms that the application is vulnerable to Mass Assignment, as sensitive attributes can be modified by user-controlled input without proper server-side validation.

## Mitigation

- Enforce server-side property-level authorization by allowing users to access or modify only the object properties they are explicitly permitted to read or update.
- Use allowlists (whitelists) to explicitly define which properties each user role can read or modify.
- Validate request payloads and reject attempts to modify protected fields such as role.
- Perform server-side authorization checks for every request, regardless of any client-side restrictions.
