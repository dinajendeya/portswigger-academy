# Lab 09: Blind SQL Injection with Time Delays

## Category

Blind SQL Injection - Time-Based SQL Injection

## Objective

Exploit a blind SQL injection vulnerability in the `TrackingId` cookie and trigger a `10-second` delay in the application's response.

---

# Solution

### 1. Understanding the Application Behavior

Let's start by visiting the shop and intercepting a request containing the `TrackingId` cookie.

Unlike the previous labs, this application does not provide any useful feedback.

I tried different payloads and noticed that:

- No data is reflected in the response.
- No error messages are displayed.
- The page looks identical regardless of whether the query succeeds or fails.

This means we cannot rely on:

- `UNION` attacks
- Error-based SQL injection
- Conditional responses such as `Welcome back`

At this point, the only thing we can potentially observe is how long the application takes to respond.

---

### 2. Testing for Time-Based SQL Injection

Since no content-based or error-based feedback exists, I wanted to see whether the database would execute a delay function.

I intercepted the request and modified the `TrackingId` cookie to:

```sql
TrackingId=x'||pg_sleep(10)--
```

The function `pg_sleep(10)` tells a PostgreSQL database to pause execution for `10` seconds before continuing.

---

### 3. Observing the Response Time

After sending the request, the application appeared to hang for approximately `10` seconds before returning the response.

The response itself looked exactly the same as before.

There was:

- No new data.
- No error message.
- No visible change in content.

The only observable difference was the response time.

This confirms that:

- The payload was executed.
- The application is vulnerable to SQL injection.
- We can use response delays as our information channel.

Since the objective of the lab is to trigger a `10-second` delay, the lab is now solved.

![The request completes after the injected time delay](../assets/SQL%20Injection/Lab%2009%20-%20Time-Based%20SQL%20Injection/step-01-trigger-time-delay.png)

### Further Testing with Intruder

After confirming the delay in Repeater, I also experimented with using Intruder to test conditional delays. I kept the attack to one thread so the responses were easier to compare.

![Setting Intruder to use one resource-pool thread](../assets/SQL%20Injection/Lab%2009%20-%20Time-Based%20SQL%20Injection/step-02-intruder-resource-pool-one-thread.png)

I then tested a condition based on the administrator password length. If the condition is true, the database waits; if it is false, it responds without the delay.

![Testing a conditional delay based on password length](../assets/SQL%20Injection/Lab%2009%20-%20Time-Based%20SQL%20Injection/step-03-password-length-delay-condition.png)

I also tried a Cluster Bomb setup to test password positions and possible characters together. This creates many requests, so the simple delay test above is enough to solve this lab.

![Cluster Bomb setup for testing password characters](../assets/SQL%20Injection/Lab%2009%20-%20Time-Based%20SQL%20Injection/step-04-cluster-bomb-password-enumeration.png)

---

# A Bit of Theory

## What is Time-Based Blind SQL Injection?

In many SQL injection labs, the application provides some form of feedback:

- Data returned in the response.
- Error messages.
- Different page content.
- Different status codes.

In this lab, none of those exist.

The application always returns the same response regardless of what happens inside the database.

As a result, we need another way to determine whether our payload was executed.

The solution is to measure time.

---

## Why Does `pg_sleep()` Work?

The payload:

```sql
pg_sleep(10)
```

tells PostgreSQL to pause execution for `10` seconds.

If our injection is successful:

```text
Request → Database sleeps 10 seconds → Response
```

If our injection fails:

```text
Request → Immediate response
```

The response content never changes, but the response time becomes our `True` / `False` signal.

---

## How Time-Based SQL Injection Is Usually Used

In this lab we only needed to create a delay.

However, in a real attack, we would use conditional delays.

For example:

```sql
SELECT CASE
WHEN (1=1)
THEN pg_sleep(10)
ELSE pg_sleep(0)
END
```

The condition `1=1` is `True`, so the database pauses for `10` seconds.

If we change it to:

```sql
SELECT CASE
WHEN (1=2)
THEN pg_sleep(10)
ELSE pg_sleep(0)
END
```

the delay never occurs because `1=2` is `False`.

This allows an attacker to ask the database a series of `True` / `False` questions and extract information one bit at a time.

---

## Comparing Blind SQL Injection Techniques

| Technique | Signal Used |
|------------|------------|
| `Conditional Response SQLi` | Different page content |
| `Conditional Error SQLi` | Presence of an error |
| `Visible Error SQLi` | Error message leaks data |
| `Time-Based SQLi` | Response delay |

This lab focuses entirely on the last technique.

---

## The Complete Process

1. Intercept the request containing `TrackingId`.
2. Observe that the application provides no useful feedback.
3. Inject `pg_sleep(10)`.
4. Send the request.
5. Observe a `10-second` delay.
6. Confirm that the payload executed successfully.

---

# What I Learned

- Identified a blind SQL injection vulnerability.
- Practiced a time-based SQL injection attack.
- Used `pg_sleep()` to introduce a controlled delay.
- Confirmed code execution without relying on page content.
- Confirmed code execution without relying on database errors.
- Learned how response time can be used as a side channel.
- Practiced identifying PostgreSQL-specific functions.

---

# Tools

- Burp Suite
- Proxy
- Repeater
- Browser

---

# Result

✅ Confirmed the blind SQL injection vulnerability.

✅ Successfully executed `pg_sleep(10)`.

✅ Caused the application response to be delayed by `10` seconds.

✅ Verified that time can be used as a side channel for extracting information.

✅ Lab solved successfully.

---

# Personal Note

This is probably the simplest time-based SQL injection lab in the Academy, but it introduces an important idea. In the previous labs we relied on error messages or different page content to determine whether our payload worked. Here we have neither. The only observable behavior is how long the application takes to respond. Once I saw the response take approximately `10` seconds after injecting `pg_sleep(10)`, it became clear that the database was executing my payload. From this point, the same technique can be extended to much more advanced labs where we use conditional delays to extract usernames, passwords, and other data one character at a time.