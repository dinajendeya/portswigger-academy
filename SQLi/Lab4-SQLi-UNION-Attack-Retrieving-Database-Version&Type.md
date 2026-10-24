# Lab 03: Examining the database in SQL injection attacks

## Category
SQL Injection - Union Attack

## Objective
perform a SQL injection UNION attack to identify the database version string.

## Solution
 
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. 
3. Verify that the query is returning two columns, both of which contain text, using such a payload in the category parameter:  
``  
'+UNION+SELECT+'abc','abc'-- 
``  
4. Use the following payload to display the database version:  
``  
'+UNION+SELECT+@@version,+'abc'-- 
``  

## A bit of theory  
To exploit SQL injection vulnerabilities, it's often necessary to find information about the database. This includes:

The type and version of the database software.
The tables and columns that the database contains.

To identify both the database type and version by injecting provider-specific queries to see if one works

| Database Type       | Query                         |
|---------------------|-------------------------------|
| Microsoft SQL Server | `SELECT @@version`           |
| MySQL               | `SELECT @@version`            |
| Oracle              | `SELECT * FROM v$version`     |
| PostgreSQL          | `SELECT version()`            |

For example, you could use a UNION attack with the following input:
``  
' UNION SELECT @@version--
``  
That would give us a result such as what we got in this lab.  

## What I Learned
- Use Burp Suite to intercept and modify the login request.
- Modify the category parameter, using Burp Suite Repeater.
- Apply SQL injection UNION attacks.
- Determine the number of columns required.
- Find columns with a useful data type.
- Use a SQL injection UNION attack to retrieve the type and version of the database software. 


## Tools
- Burp Suite
- Browser

## Result
✅ Retrieve the type and version of the database software.  
✅ Lab solved successfully

