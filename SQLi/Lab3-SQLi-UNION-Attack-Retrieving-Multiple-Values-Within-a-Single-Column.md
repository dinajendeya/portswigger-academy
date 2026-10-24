# Lab 03: Retriving multiple values within a single column

## Category
SQL Injection - Union Attack

## Objective
perform a SQL injection UNION attack that retrieves all usernames and passwords within a single column, and use the information to log in as the administrator user.

## Solution
 
1. Use Burp Suite to intercept and modify the request that sets the product category filter.

2. Determine the number of columns that are being returned by the query and which columns contain text data. 

3. Verify that the query is returning two columns, only one of which contain text, using a payload like the following in the category parameter:  

``
 '+UNION+SELECT+NULL,'abc'-- 
 ``  

4. Use the following payload to retrieve the contents of the users table:  

``
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
``  

5. Verify that the application's response contains usernames and passwords.

6. login as an administrator user, using the data you retrive from the previous step.


## A bit of theory  
In some cases the query may only return a single column.

You can retrieve multiple values together within this single column by concatenating the values together. You can include a separator to let you distinguish the combined values. For example, on Oracle you could submit the input:
``  
' UNION SELECT username || '~' || password FROM users-- 
`` 
This uses the double-pipe sequence || which is a string concatenation operator on Oracle. The injected query concatenates together the values of the username and password fields, separated by the ~ character. Like the outcome of this lab.

Different databases use different syntax to perform string concatenation.  

## What I Learned
- Use Burp Suite to intercept and modify the login request.
- Modify the category parameter, using Burp Suite Repeater.
- Apply SQL injection UNION attacks.
- Determine the number of columns required.
- Find columns with a useful data type.
- Use a SQL injection UNION attack to retrieve multiple interesting values within a single column.


## Tools
- Burp Suite
- Browser

## Result
✅ Unautherized login as any eligible user.  
✅ Lab solved successfully

