---
title: "AWS Cloud Enumeration Command"
categories:
- Cloud Security Penetration Testing
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2025-09-21-aws-cloud-enumeration-command
tags:
- AWS Pentesting
- AWS Cloud Pentesting
- S3 Bucket Pentesting
- Lambda Pentesting
- AWS Enumeration
- AWS Command
- AWSCLI Command
- Pacu
- web vulnerability
- ethical hacking
- bug bounty
- application security
---

## Creating Profile

### a. Create the aws profile

```
aws configure --profile <profile-name>

```

### b. Verify access

```
aws sts get-caller-identity --profile <profile-name>

```

## User Enumeration

### a. Get user information

```
aws iam get-user --profile <profile-name>
```

### b. List all IAM users

```
aws iam list-users --profile <profile-name>
aws iam list-users

```

### c. List Access Key

```
aws iam list-access-keys --profile <profile-name>

```

## Get User Permissions

### a. List attached managed policies

```
aws iam list-attached-user-policies --user-name [user-name]
aws iam list-attached-user-policies --user-name [user-name] --profile <profile-name>
```

### b. List inline policies

```
aws iam list-user-policies --user-name [user-name]
aws iam list-user-policies --user-name [user-name] --profile <profile-name>

```

### c. Get inline policy details

```
aws iam get-user-policy --user-name [user-name] --policy-name [policy-name]
```

## List IAM Groups and Permissions

### a. List all IAM Groups

```
aws iam list-groups --profile <profile-name>
```

### b. List assigned groups for a user

```
aws iam list-groups-for-user --user-name [user-name]
aws iam list-groups-for-user --user-name [user-name] --profile <profile-name>
```

### c. List group policies

```
aws iam list-attached-group-policies --group-name [group-name]
aws iam list-group-policies --group-name [group-name]
```

### d. Get inline group policy details

```
aws iam get-group-policy --group-name [group-name] --policy-name [policy-name]
```

## List IAM Roles and Permissions

### a. List all roles**

```
aws iam list-roles
aws iam list-roles --profile <profile-name>
```

### b. Get role details (trust policy)

```
aws iam get-role --role-name [role-name]
```

### c. List attached policies

```
aws iam list-attached-role-policies --role-name [role-name]
```

### d. List inline policies

```
aws iam list-role-policies --role-name [role-name]
```

### e. Get inline role policy details

```
aws iam get-role-policy --role-name [role-name] --policy-name [policy-name]
```

## Get and Decode Policy Documents

### a. Get a managed policy document (by ARN or name)

```
aws iam get-policy --policy-arn [policy-arn]
aws iam get-policy-version --policy-arn [policy-arn] --version-id [version-id]
```

## View Full IAM Snapshot

### a. Dump all IAM permissions (users, roles, groups, policies)

```
aws iam get-account-authorization-details
```

## S3 buckets Enumeration

### a. List the bucket

```
aws s3 ls --profile <profile-name>
```

### b. View s3 bucket data

```
aws s3 ls s3://<s3-bucket-name> --profile <profile-name>
```

### c. Copy s3 bucket

```
aws s3 cp s3://<s3-bucket-name>/<file-name> . --profile <profile-name>
```

## Ec2 Enumeration

### a. List running instances

```
aws ec2 describe-instances --region <region> --profile <profile-name>
```

### b. List security groups of identified instances

```
aws ec2 describe-security-groups --group-ids <group-id > --region  <region> --profile <profile-name>
```

### c. Enumerate permission of the instance profile

```
aws iam get-instance-profile --instance-profile-name <profile-name> --region <region> --profile <profile-name>
```

### d. Check user data for the EC2 instance

```
aws ec2 describe-instance-attribute --instance-id <instance-id> --attribute userData --region <region> --profile <profile-name>
```

### e. Check metadata

```
curl -s http://169.254.169.254/latest/meta-data/
```

## Lambda Enumeration

### a. List Lambda function

```
aws lambda list-functions --region <region> --profile <profile-name>
```

### b. Get details of a specific lambda function

```
aws lambda get-function --function-name <FUNCTION_NAME>
```

## Secret Manager Enumeration

### a. List secret manager

```
aws secretsmanager list-secrets --region <region> --profile <profile-name>
```

Continue.......
