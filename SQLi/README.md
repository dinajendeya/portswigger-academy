<div align="center">

# SQL Injection: My PortSwigger Lab Notes

**Learning SQL injection by testing it in the Academy, writing down what happened, and thinking about how to prevent it.**

![Focus](https://img.shields.io/badge/focus-web%20application%20security-red)
![Platform](https://img.shields.io/badge/platform-PortSwigger%20Academy-orange)
![Labs](https://img.shields.io/badge/labs%20solved-16%2B-blue)
![Status](https://img.shields.io/badge/status-comleted-brightgreen)
![Studying](https://img.shields.io/badge/studying-SQL%20injection-lightgrey)

[Introduction](#introduction) · [Lab references](#lab-references) · [Skills](#what-i-learned) · [Developer notes](#what-developers-can-do) · [Connect](#connect-with-me)

</div>

---

## Introduction

SQL injection was one of the first topics I wanted to understand by actually working through vulnerable applications. So far, I have solved around 16 SQL injection labs in the [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection). I chose 10 of them for this folder because together they show techniques that helped me understand the topic from different angles.

These are my learning notes, not a list of payloads to copy blindly. Each walkthrough explains what I tried, what the response showed me, and what I learned along the way. I added screenshots where I captured them, and included the matching Academy reference for each lab.

## Lab references

| Lab | Technique I practised | My walkthrough | PortSwigger reference |
|---|---|---|---|
| Lab 01 | Login bypass and SQL comments | [Read my notes](./Lab01-SQL-Injection-Login-Bypass.md) | [Login bypass lab](https://portswigger.net/web-security/sql-injection/lab-login-bypass) |
| Lab 02 | `UNION` attack to retrieve data | [Read my notes](./Lab02-SQLi-UNION-Attack-to-Retrieve-Interesting-Data.md) | [Retrieve hidden data lab](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data) |
| Lab 03 | Combining values in one `UNION` column | [Read my notes](./Lab03-SQLi-UNION-Attack-Retrieving-Multiple-Values-Within-a-Single-Column.md) | [Retrieve multiple values in one column lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column) |
| Lab 04 | Identifying database type and version | [Read my notes](./Lab04-SQLi-UNION-Attack-Retrieving-Database-Version&Type.md) | [UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks) |
| Lab 05 | Inspecting tables and columns | [Read my notes](./Lab05-SQLi-UNION-Attack-Retrieving-Database-Schema.md) | [UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks) |
| Lab 06 | Blind SQLi with conditional responses | [Read my notes](./Lab06-SQLi-Conditional-Response.md) | [Conditional responses lab](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses) |
| Lab 07 | Blind SQLi with conditional errors | [Read my notes](./Lab07-SQLi-Conditional-Response-Error-Based.md) | [Conditional errors lab](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors) |
| Lab 08 | Visible errors that reveal query data | [Read my notes](./Lab08-SQLi-Visible-Error-Based.md) | [SQL injection](https://portswigger.net/web-security/sql-injection) |
| Lab 09 | Blind SQLi using time delays | [Read my notes](./Lab09-SQL-Injection-Time-Delay.md) | [Time delays lab](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays) |
| Lab 10 | XML filter bypass and `UNION` | [Read my notes](./Lab10-SQL-Injection-XML-Filter-Bypass.md) | [XML encoding filter-bypass lab](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding) |

The walkthroughs include screenshots placed beside the steps they explain. Each write-up also has its own explanation of the result and what I learned.

## What I learned

Working through these labs helped me practise how to:

- Spot when input changes the way an SQL query behaves.
- Use `UNION` queries and figure out the number and types of returned columns.
- Learn about database versions, tables, and columns from query results.
- Recognize that a vulnerable login form can change the meaning of an authentication query.
- Ask true-or-false questions when the application does not show database results directly.
- Notice when database errors or response times reveal information.
- Use Burp Suite Repeater and Intruder to test ideas and compare responses.
- Keep screenshots and notes so I can follow my own steps again later.

## What developers can do

The main lesson I took from these labs is that input should not become part of a SQL command. For developers, the most reliable first step is to use **parameterized queries or prepared statements**, so values stay data instead of changing the query structure.

It also helps to:

- Avoid building SQL by joining user input into query strings.
- Use database accounts with only the permissions the application needs.
- Return general error messages to users, while keeping detailed diagnostics in protected server logs.
- Validate input for the expected format, while treating validation as an extra check rather than a replacement for parameterized queries.
- Test database-facing features with security checks and regression tests.
- Avoid relying on keyword filters or a WAF as the main protection against SQL injection.

## Connect with me

I'm still learning, so feedback, corrections, and advice are welcome.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-connect-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/dina-jendeya-255207256)
[![GitHub](https://img.shields.io/badge/GitHub-follow-181717?logo=github&logoColor=white)](https://github.com/dinajendeya)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-profile-212C42?logo=tryhackme&logoColor=white)](https://tryhackme.com/p/dinajendeya)
[![Email](https://img.shields.io/badge/Email-contact-D14836?logo=gmail&logoColor=white)](mailto:deenayaseer@gmail.com)

---

[Back to the repository README](../README.md)
