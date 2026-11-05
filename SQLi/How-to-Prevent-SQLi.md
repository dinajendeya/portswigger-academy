# Preventing SQL Injection in Real Applications

## Category

Secure Coding - SQL Injection Prevention

## Objective

Understand how to properly prevent `SQL Injection` vulnerabilities using `Parameterized Queries` and `Whitelisting`, learn why these techniques work under the hood, and see how they are implemented in a real-world `TypeScript`, `Node.js`, and `PostgreSQL` application as those are my prefered tech stack.

---

# Introduction

After solving many SQL Injection labs in PortSwigger Academy, I started noticing that almost every successful SQL Injection attack exists because the application allows user input to become part of the SQL query itself.

As a software engineer, I am not interested only in exploiting vulnerabilities. I also want to understand how to prevent them correctly.

The two most important defenses are:

1. `Parameterized Queries`
2. `Whitelisting`

When implemented properly, these techniques prevent most SQL Injection vulnerabilities before they even reach the database.

---

# The Root Cause of SQL Injection

Let's imagine we have a login page where the user enters an email address.

A beginner developer might write:

```ts
const email = req.body.email;

const query = `
SELECT *
FROM users
WHERE email = '${email}'
`;
```

At first glance, this looks fine. However, the application is directly inserting user input into the SQL statement.

Suppose an attacker submits: ``
' OR 1=1 --
``

The final query becomes:

```sql
SELECT *
FROM users
WHERE email = '' OR 1=1 --'
```

The condition `1=1` is always `TRUE`.

As a result, the database returns every row in the table.

The real problem is that the database cannot distinguish between:

- application code
- attacker-controlled data

Everything becomes part of one SQL statement.

---

# The Best Defense: Parameterized Queries

Instead of inserting user data directly into the query string, we send the SQL code and the user values separately. For example:

```ts
const email = req.body.email;

const query = `
SELECT *
FROM users
WHERE email = $1
`;

const result = await pool.query(query, [email]);
```

This is the approach used by the PostgreSQL package `pg`.

Notice the placeholder: ``
$1
``. This does not mean "the value itself".

It means: ``
Put the first parameter here.
``

The value is supplied separately: ``
[email]
``

---

## What Happens Behind the Scenes?

This is the part that many people miss.

Suppose the user submits:
``
admin@example.com
``

The database receives:

```sql
SELECT *
FROM users
WHERE email = $1
```

and separately receives:
``
admin@example.com
``

The database engine knows: the first one is ``SQL code.`` and the second one is the ``user data`` now the user value never becomes executable SQL.

---

## What If an Attacker Tries SQL Injection?

Suppose the attacker submits:
``
' OR 1=1 --
``

The query remains:

```sql
SELECT *
FROM users
WHERE email = $1
```

The database treats the attack payload as ordinary text:
``
' OR 1=1 --
`` It does **not** treat it as SQL code.

Internally, PostgreSQL sees:

```text
Find a user whose email literally equals:
' OR 1=1 --
```

That record obviously does not exist. The query simply returns:
``
0 rows
``. The SQL Injection fails completely.

---

# Using Prisma ORM

Modern applications often use an ORM such as `Prisma`.

Consider:

```ts
const user = await prisma.user.findUnique({
  where: {
    email: req.body.email
  }
});
```

This is even safer because Prisma builds the SQL query for us. The developer never manually creates SQL strings. Prisma internally generates parameterized queries before sending them to PostgreSQL. ***As a result:***

```text
User Input
↓
Prisma
↓
Parameterized Query
↓
PostgreSQL
```

No SQL Injection occurs.

---

# Safe Raw Queries in Prisma

Sometimes we need raw SQL means a literal SQL query write by the developer without using the built in function maybe becuase this query is more sophisticated than the usual database usage like update, delete, create or insert.

Prisma provides:

```ts
const users = await prisma.$queryRaw`
SELECT *
FROM users
WHERE email = ${email}
`;
```

Many beginners think this looks dangerous. In reality, Prisma parameterizes the value automatically.

The variable:
``
${email}
`` is not copied directly into the query string. Prisma safely converts it into parameters behind the scenes.

Therefore: it is ***safe***

---

# Unsafe Raw Queries

Now look at this example:

```ts
const query =
`SELECT * FROM users WHERE email = '${email}'`;

await prisma.$queryRawUnsafe(query);
```

This is completely different. The SQL string is built manually. If an attacker controls:
``
email
`` they control part of the SQL statement.

This is exactly how SQL Injection vulnerabilities happen.

Therefore: it is ***unsafe***

---

# The Second Defense: Whitelisting

Parameterized queries protect values.

However, some SQL elements cannot be parameterized.

Examples include:

- table names
- column names
- sort directions
- `ASC`
- `DESC`

For instance:

```sql
ORDER BY ?
```

is not valid SQL.

The database expects a column name, not a parameter. Developers sometimes make this mistake:

```ts
const order = req.query.order;

const query = `
SELECT *
FROM users
ORDER BY created_at ${order}
`;
```

An attacker could submit:
``
DESC; DROP TABLE users --
``. Now the application becomes dangerous.

---

# Solving This with Whitelisting

Instead of trusting the user input, we define acceptable values. For example:

```ts
const order =
req.query.order === "asc"
  ? "ASC"
  : "DESC";
```

The query becomes:

```ts
const query = `
SELECT *
FROM users
ORDER BY created_at ${order}
`;
```

Now only two values are possible:
``
ASC
``
``
DESC
``.
Nothing else will ever reach the SQL statement.

Even if the attacker submits:
``
DROP TABLE users
`` after the expected value the application either reject it or conver it safely to
``
DESC
``

---

# Real-World Example

Imagine an admin dashboard. The developer wants users to sort accounts by:

- username
- email
- creation date

A dangerous solution would be:

```ts
const sortBy = req.query.sort;

const query = `
SELECT *
FROM users
ORDER BY ${sortBy}
`;
```

Instead, use a whitelist:

```ts
const allowedColumns = {
  username: "username",
  email: "email",
  created: "created_at"
};

const sortColumn =
allowedColumns[req.query.sort] ??
"created_at";
```

Now only approved columns can be used. Anything unexpected falls back to:
``
created_at
``, which completely removes the attack surface.

---

# Combining Both Defenses

The safest applications use both techniques together. For example:

```ts
const sortColumn =
allowedColumns[req.query.sort]
?? "created_at";

const query = `
SELECT *
FROM users
WHERE email = $1
ORDER BY ${sortColumn}
`;

await pool.query(query, [email]);
```

Notice what happened:

- The value `email` uses parameterization.
- The column name uses whitelisting.

This is exactly how secure production applications are commonly built.

---

# Common Mistakes Developers Make

### 1. Trusting Input Validation Alone

Some developers write:

```ts
if (!email.includes("'")) {
  // safe
}
```

This is not security. Attackers constantly find new bypasses. Always use parameterized queries.

---

### 2. Using Regex Instead of Parameterization
Example:

```ts
const safeEmail =
email.replace(/'/g, "");
```

This is weak protection.

You are attempting to clean attacker input rather than properly separating code from data.

---

### 3. Building Dynamic SQL Everywhere

Example:

```ts
const query =
`SELECT * FROM users WHERE id = ${id}`;
```

This becomes dangerous very quickly. Instead use placeholders such as:
``
$1
``.

---

# What I Learned

- SQL Injection exists because user input becomes part of SQL code.
- Parameterized queries separate SQL code from user data.
- PostgreSQL placeholders such as `$1`, `$2`, and `$3` prevent user data from becoming executable SQL.
- Prisma automatically parameterizes values in most scenarios.
- `queryRawUnsafe()` should be avoided whenever possible.
- Some SQL parts such as table names and `ORDER BY` values cannot be parameterized.
- Whitelisting should be used whenever parameterization is impossible.
- Combining parameterized queries and whitelisting provides strong protection against SQL Injection.
- Secure coding is far more effective than trying to filter malicious payloads.

---

# Recommended Approach

For a modern `Node.js`, `TypeScript`, and `PostgreSQL` application:

```text
User Values
→ Parameterized Queries

Column Names
→ Whitelist

Table Names
→ Whitelist

Sort Directions
→ Whitelist

Raw SQL
→ Avoid if possible

Prisma ORM
→ Prefer whenever practical
```

---

# Personal Note

After completing many SQL Injection labs, I noticed that most successful attacks eventually come back to the same developer mistake: mixing SQL code with user input. The exact payload may change from `' OR 1=1 --` to a blind attack using `TRUE`, `FALSE`, errors, or even `pg_sleep(10)`, but the root cause remains the same. That is why I now think of `Parameterized Queries` as the primary defense and `Whitelisting` as the secondary defense. If user data never becomes SQL code, most SQL Injection attacks simply stop working before they even begin.