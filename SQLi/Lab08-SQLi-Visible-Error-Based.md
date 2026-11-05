# Lab 08: Visible Error-Based SQL Injection

## Category

Error-Based SQL Injection

## Objective

Exploit a SQL injection vulnerability in the `TrackingId` cookie and use database error messages to leak the password of the `administrator` user, then log in as that user.

---

# Solution

### 1. Identifying the Injection Point

Let's start by browsing the application and intercepting a request containing the `TrackingId` cookie.

For simplicity, assume the original cookie value looks like:

```http
TrackingId=ogAZZfxtOKUELbuJ
```

To test whether the parameter is vulnerable, I appended a single quote and changed it to:

```http
TrackingId=ogAZZfxtOKUELbuJ'
```

After sending the request, the application returned a verbose error message.

What immediately caught my attention was that the response leaked part of the SQL query and mentioned an unclosed string literal.

This is a very strong indicator that our input is being placed inside a SQL query and that the application is vulnerable to SQL injection.

![A single quote returns an unclosed string error](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-01-single-quote-database-error.png)

---

### 2. Fixing the Syntax Error

Since the error indicated an unclosed string, I wanted to see whether I could terminate the query correctly.

I modified the cookie to:

```sql
TrackingId=ogAZZfxtOKUELbuJ'--
```

The comment sequence `--` causes the remainder of the query to be ignored.

After sending the request again, the error disappeared.

This suggests that our payload is now producing valid SQL syntax.

![Adding a SQL comment fixes the syntax error](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-02-comment-fixes-syntax.png)

---

### 3. Testing a Simple Query

Now I wanted to see whether I could inject my own query.

I changed the cookie to:

```sql
TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
```

A new error appeared saying that the `AND` condition must be a boolean expression.

This makes sense because:

```sql
CAST((SELECT 1) AS int)
```

returns an integer and not a `True` or `False` value.

![The database reports that the injected value is not boolean](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-03-cast-returns-integer-error.png)

---

### 4. Making the Condition Boolean

To fix this, I added a comparison operator and changed the payload to:

```sql
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
```

The error disappeared.

This tells us that: ``
1=1
``

evaluates to `True`, making the entire query valid.

At this point we know that:

- SQL injection works.
- We can execute subqueries.
- We can use `CAST()`.
- Error messages are visible.

![Adding a comparison makes the injected condition valid](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-04-boolean-condition-works.png)

---

### 5. Attempting to Extract Usernames

The next step was to retrieve data from the database.

I changed the payload to:

```sql
TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
```

The response returned another error.

However, I noticed that part of the payload appeared to be truncated.

After looking more closely, I realized the value of the original `TrackingId` was consuming valuable space, making the payload too long.

![The original cookie value leaves too little room for the query](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-05-original-cookie-limits-payload.png)

---

### 6. Removing the Original Cookie Value

To free up additional characters, I removed the original value completely and changed the cookie to:

```sql
TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
```

This time the error changed.

Instead of a syntax error, the database complained that the query returned more than one row.

This is actually good news.

It means:

- The query executed successfully.
- The table `users` exists.
- The column `username` exists.
- Multiple usernames were returned.

![The query returns more than one row](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-06-subquery-returns-multiple-rows.png)

---

### 7. Limiting the Result to One Row

To fix the multiple-row issue, I added `LIMIT 1`:

```sql
TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

After sending the request, the response returned a database conversion error:

```text
ERROR: invalid input syntax for type integer: "administrator"
```

This happened because the database attempted to execute:

```sql
CAST('administrator' AS int)
```

which is impossible.

The important part is that the username is leaked directly inside the error message.

This confirms that the first username in the table is:

```text
administrator
```

![The database error reveals the administrator username](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-07-administrator-username-leaked.png)

---

### 8. Extracting the Password

Now that we know the first row belongs to the `administrator` user, we can use exactly the same technique to retrieve the password.

I also tried checking whether the password was longer than one character in Repeater. This was another useful test, but the conversion error below let me retrieve the full value directly.

![Testing the administrator password length in Repeater](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-08-password-length-query.png)

I modified the payload to:

```sql
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

After sending the request, another conversion error appeared.

This time the database attempted:

```sql
CAST('<administrator-password>' AS int)
```

Since a password cannot be converted to an integer, the contents of the password were leaked directly inside the error message.

![The database error reveals the administrator password](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-09-administrator-password-leaked.png)

Unlike blind SQL injection labs where we extract one character at a time, this lab allows us to retrieve the entire password in a single request.

---

### 9. Logging In

After recovering the password, I clicked **My Account** and logged in using:

- Username: `administrator`
- Password: `<retrieved-password>`

The lab was solved successfully.

![Administrator login succeeds and the lab is solved](../assets/SQL%20Injection/Lab%2008%20-%20Visible%20Error-Based%20SQL%20Injection/step-10-lab-solved.png)

---

# A Bit of Theory

## What is Error-Based SQL Injection?

Error-based SQL injection occurs when:

1. User input reaches a SQL query.
2. The application displays database errors.
3. Those errors reveal useful information.

Instead of extracting data through the page content, we intentionally cause database errors that expose information.

---

## Why Does `CAST()` Leak Data?

Consider the following query:

```sql
SELECT username FROM users LIMIT 1
```

If the result is: ``
administrator
``

and we try:

```sql
CAST('administrator' AS int)
```

the database cannot convert text into an integer.

As a result, it throws an error similar to:

```text
ERROR: invalid input syntax for type integer: "administrator"
```

The error itself leaks the value we wanted to retrieve.

---

## Comparing Blind and Error-Based SQL Injection

| Type | Data Retrieval |
|--------|--------|
| `Blind SQLi` | Infer data indirectly using `True` / `False` conditions |
| `Time-Based SQLi` | Infer data using response delays |
| `Error-Based SQLi` | Extract data directly from database errors |

The reason this lab is easier than most blind SQL injection labs is because the database reveals the actual data inside its error messages.

---

## The Complete Process

1. Identify the injectable parameter.
2. Trigger a SQL syntax error.
3. Comment out the remaining query.
4. Verify execution using `CAST()`.
5. Query the `users` table.
6. Force a conversion error.
7. Leak the username.
8. Leak the password.
9. Log in as `administrator`.

---

# What I Learned

- Identified a SQL injection vulnerability in the `TrackingId` cookie.
- Used database error messages to understand the SQL query structure.
- Used `--` to comment out the remainder of a query.
- Used `CAST()` to generate controlled database errors.
- Confirmed the existence of the `users` table.
- Extracted the `administrator` username from an error message.
- Extracted the administrator password from an error message.
- Practiced visible error-based SQL injection.
- Retrieved sensitive information without using `UNION` attacks.

---

# Tools

- Burp Suite
- Proxy
- HTTP History
- Repeater
- Browser

---

# Result

✅ Confirmed the SQL injection vulnerability.

✅ Determined that verbose database errors were exposed to users.

✅ Verified the existence of the `users` table.

✅ Leaked the username `administrator`.

✅ Leaked the administrator password using a `CAST()` conversion error.

✅ Successfully authenticated as `administrator`.

✅ Lab solved successfully.

---

# Personal Note

Compared to the previous blind SQL injection labs, this one felt much easier because the database practically tells us the answer through its error messages. Instead of asking hundreds of `True` / `False` questions or running Intruder attacks, we can force a data type conversion error and let the database reveal the value for us. This is a good reminder that detailed error messages can be just as dangerous as a direct SQL injection vulnerability.