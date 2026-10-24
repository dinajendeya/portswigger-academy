# Lab: SQL injection vulnerability allowing login bypass

## Category
SQL Injection

## Objective
perform a SQL injection attack that logs in to the application as the administrator user.

## Solution
1. Use Burp Suite to intercept and modify the login request.

2. Modify the username parameter, giving it the value: 
``
administrator'-- 
``
## A bit of theory  
If a user submits the username administrator and the password password, the application checks the credentials by performing the following SQL query:

``
    SELECT * FROM users WHERE username = 'USER_NAME' AND password = 'PASSWORD'
``  
If the query returns the details of a user, then the login is successful. Otherwise, it is rejected.

In this case, we can log in as any user without the need for a password. we can do this using the SQL comment sequence -- to remove the password check from the WHERE clause of the query. For example, submitting the username administrator'-- and a blank password results in the following query:

SELECT * FROM users WHERE username = 'administrator'--' AND password = ''

This query returns the user whose username is administrator and successfully logs the attacker in as that user.



## What I Learned
- Use Burp Suite to intercept and modify the login request.
- Modify the username parameter, giving it the value: administrator'-- 


## Tools
- Burp Suite
- Browser

## Result
✅ Unautherized login as an administrator user.  
✅ Lab solved successfully

