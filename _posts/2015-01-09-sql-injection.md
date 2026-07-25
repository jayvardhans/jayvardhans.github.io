---
title: "SQL Injection"
categories:
- OWASP Top 10 Application Vulnerability
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2015-01-09-sql-injection
tags:
- SQL Injection
- Union based SQL Injection
- Error Based SQL Injection
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## What is SQL Injection Attack?

SQL injection is a web application vulnerability in which an attacker injects malicious SQL statements through user input. If the application fails to properly validate or sanitize the input, these statements can be executed by the database, potentially allowing unauthorized access, data modification, or data theft.

## Type of SQL Injection

SQL injection attacks can be categorized based on how the attacker interacts with the application and database. The three main types are:

## 1- In-Band SQL Injection

This is the most common type, where the attacker sends malicious SQL query and receive the results. 

  ## a- Error-Based SQL Injection: 
  
  The attacker exploits database error messages to gather information about the database structure.

  ## b- Union-Based SQL Injection: 
  
  The attacker uses the UNION SQL operator to combine the results of a malicious query with the application's original query and retrieve data from other tables.

## 2- Inferential (Blind) SQL Injection

In Blind SQL injection, the application does not display database errors or query results directly. Instead, the attacker infers information by observing the application's behavior.

  ## a- Boolean-Based Blind SQL Injection: 
  
  The attacker sends queries that evaluate to TRUE or FALSE and determines information based on differences in the application's response.

  ## b- Time-Based Blind SQL Injection: 
  
  The attacker uses database delay functions (such as SLEEP() or WAITFOR DELAY) and observes response times to infer whether a condition is true.

## 3- Out-of-Band SQL Injection

Out-of-Band SQL injection occurs when the attacker cannot retrieve data through the application's normal response. Instead, the database sends information through a different communication channel, such as DNS or HTTP requests. This technique depends on database features that support external network communication.
