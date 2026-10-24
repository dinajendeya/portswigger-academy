# Lab 02:Using a SQL injection UNION attack to retrieve interesting data

## Category
SQL Injection - Union Attack

## Objective
perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user.

## Solution
 
1. Use Burp Suite to intercept and modify the request that sets the product category filter.

2. Determine the number of columns that are being returned by the query and which columns contain text data. 

3. Determine which columns contain text data, using a payload like the following in the category parameter:  

``
 '+UNION+SELECT+NULL,'abc'-- 
 ``  

4. Use the following payload to retrieve the contents of the users table:  

``
'+UNION+SELECT+username,+password+FROM+users--
``  

5. Verify that the application's response contains usernames and passwords.

6. login as an administrator user, using the data you retrive from the previous step.

## A bit of theory  
When an application is vulnerable to SQL injection, and the results of the query are returned within the application's responses, you can use the UNION keyword to retrieve data from other tables within the database. This is commonly known as a SQL injection UNION attack.

For a UNION query to work, two key requirements must be met:

1. The individual queries must return the same number of columns.

2. The data types in each column must be compatible between the individual queries.

The UNION keyword enables you to execute one or more additional SELECT queries and append the results to the original query. For example:  
``  
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
``  
This SQL query returns a single result set with two columns, containing values from columns a and b in table1 and columns c and d in table2. 

To determine the number of columes there are two main ways: 
1. Injecting a serise of order by specifying the id number of the colums. If exceeded the right number the page will responed with an error. For example, if the injection point is a quoted string within the WHERE clause of the original query, you would submit:
``  
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
``  
etc.

2. Modify the injected parameter, giving it the value 
``  
'+UNION+SELECT+NULL-- 
``  
continue adding additional columns containing a null value untill there is no errors. That means we found the right number of columns
``  
'+UNION+SELECT+NULL,NULL--
``  
We used the NULL type because is easy to convert to other data type without releasing an error.
To test the type of data being returned withing the original queries we can replace the null with a value of the interested data type. For example:  
``  
'+UNION+SELECT+'ABC',NULL,NULL-- 
'+UNION+SELECT+NULL,'ABC',NULL-- 
'+UNION+SELECT+NULL,NULL,'ABC'-- 
``  
Once we found the right number of columns with the interested data type we can perform the SQL injection union attack successfully.


## What I Learned
- Use Burp Suite to intercept and modify the login request.
- Modify the category parameter, using Burp Suite Repeater.
- Apply SQL injection UNION attacks.
- Determine the number of columns required.
- Find columns with a useful data type.
- Use a SQL injection UNION attack to retrieve interesting data.


## Tools
- Burp Suite
- Browser

## Result
✅ Unautherized login as any eligible user.  
✅ Lab solved successfully

