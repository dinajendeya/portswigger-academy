# Lab 01: SQL injection vulnerability allowing login bypass

## Category
SQL Injection

## Objective
perform a SQL injection attack that logs in to the application as the administrator user.

## Solution
1. Use Burp Suite to intercept and modify the login request.

![The intercepted login request and its response](../assets/SQL%20Injection/Lab%2001%20-%20Login%20Bypass/step-01-intercept-login-request.png)

I sent the request to Repeater so I could change the username and test the login again.

![Sending the login request to Repeater](../assets/SQL%20Injection/Lab%2001%20-%20Login%20Bypass/step-02-send-request-to-repeater.png)

2. Modify the username parameter, giving it the value: 
``
administrator'-- 
``

I left the password field filled in, but the comment sequence makes the application ignore the password check in this vulnerable query.

![Submitting the username with the SQL comment sequence](../assets/SQL%20Injection/Lab%2001%20-%20Login%20Bypass/step-03-submit-login-bypass-payload.png)

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

![The administrator account is logged in and the lab is solved](../assets/SQL%20Injection/Lab%2001%20-%20Login%20Bypass/step-04-administrator-login-success.png)
