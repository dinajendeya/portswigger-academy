<div align="center">

# 🔐 PortSwigger Web Security Academy: My Learning Journey

**Learning offensive security by doing, and documenting every step in public.**

![Focus](https://img.shields.io/badge/focus-web%20application%20security-red)
![Platform](https://img.shields.io/badge/platform-PortSwigger%20Academy-orange)
![Labs](https://img.shields.io/badge/labs%20solved-0%2F60-blue)
![Status](https://img.shields.io/badge/status-actively%20learning-brightgreen)
![Studying](https://img.shields.io/badge/studying-Security%2B%20%7C%20CCNA-lightgrey)

[About Me](#-about-me) · [Why This Repo](#-why-this-repository) · [Structure](#-repository-structure) · [Template](#-walkthrough-template) · [Progress](#-progress-tracker) · [Connect](#-connect-with-me)

</div>

---

## 👋 Introduction

This repository is my knowledge base and progress tracker for the [PortSwigger Web Security Academy](https://portswigger.net/web-security).
For every lab I solve, I write down **how I approached it, what I got wrong, what I learned, and how a developer would fix it**.

I'm not here to show off a finished skill set. I'm here to build one, in the open, with discipline and consistency.

## 🙋 About Me

I'm **Dina**, a **Software Engineer** from **Gaza**, graduating with a **GPA of 90.66**.
My focus is **cybersecurity**, especially **web application security and penetration testing**.

Right now I'm:

- 🧪 Practising hands-on on **TryHackMe**, **OverTheWire** and **PortSwigger Academy**
- 🛠️ Building backend projects, so I understand how the things I'm learning to break are actually built
- ✍️ Documenting what I learn, because explaining something is the fastest way to find out whether I really understand it

## 💡 Why This Repository?

- **Learning by doing.** Reading about SQL injection is not the same as exploiting it in a lab and then fixing the vulnerable code.
- **Building the habit.** A public repo keeps me consistent. Small, regular progress beats occasional bursts.
- **Turning labs into understanding.** Solving a lab is step one. Writing down *why* it worked is where the learning sticks.
- **A knowledge base for later.** In a year I want to open this repo and see how far I've come.
- **Helping other learners.** If my notes save one beginner an hour of confusion, they were worth writing.

## 🎯 Learning Objectives

- [ ] Understand the most common web vulnerabilities **at the root cause level**, not just the payload
- [ ] Build a repeatable methodology: recon → hypothesis → test → exploit → document
- [ ] Get comfortable with **Burp Suite** (Proxy, Repeater, Intruder and friends)
- [ ] Learn to think like both an attacker **and** a developer who has to fix the bug
- [ ] Write clear, honest, reproducible technical notes
- [ ] Complete **60+ labs** across the core vulnerability categories

## 🧰 Skills I'm Developing

| Area | What I'm practising |
|---|---|
| 🌐 Web application security | SQL injection, authentication flaws, access control, CSRF, XSS, JWT, SSRF, file upload, business logic, race conditions |
| 🕵️ Penetration testing mindset | Methodical testing, forming and checking hypotheses, patience when stuck |
| 🔧 Tooling | Burp Suite, browser dev tools, cURL, Postman |
| 🛡️ Secure coding | Reading vulnerable code, writing fixes, adding regression tests |
| 🌍 Networking and fundamentals | HTTP in depth, TCP/IP, and CCNA topics that make web attacks easier to reason about |
| 📝 Technical communication | Concise write-ups other people can follow |

## 📂 Repository Structure

```
portswigger-writeups/
├── README.md
├── templates/
│   └── walkthrough-template.md
├── sql-injection/
│   ├── README.md                 # category overview + lessons learned
│   └── 01-where-clause-hidden-data.md
├── authentication/
├── access-control/
├── csrf/
├── xss/
├── jwt/
├── ssrf/
├── file-upload/
├── business-logic/
├── race-conditions/
└── resources/                    # cheat sheets, tool notes, reading list
```

Each category folder has its own `README.md` with the big-picture lessons, and one file per lab.
File names follow `NN-short-lab-name.md`.

## 📝 Walkthrough Template

Every lab write-up follows the same structure, so the notes stay consistent and easy to review.

```markdown
# Lab: <exact lab title>

| | |
|---|---|
| **Category** | e.g. SQL injection |
| **Difficulty** | Apprentice / Practitioner / Expert |
| **Lab link** | <link> |
| **Date solved** | YYYY-MM-DD |
| **Tools used** | Burp Suite Proxy, Repeater |

## 🎯 Goal
One or two sentences on what the lab asks for.

## 🧠 Concept
The vulnerability in plain language, as if explaining it to a classmate.

## 🔎 My approach
1. What I noticed first
2. What I tried and why
3. What didn't work, and what that taught me

## 💥 Exploitation (high level)
The key idea of the exploit and the evidence that confirmed it.

## 🧱 Root cause
Why the application was vulnerable.

## 🛡️ How to fix it
What a developer should change, with a short code example where useful.

## 📌 Lessons learned
- What I'd check first next time
- What I misunderstood before this lab

## 🔗 References
- PortSwigger learning path, OWASP page, etc.
```

### ✏️ Example entry (format only)

> **Lab:** SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
> **Difficulty:** Apprentice · **Tools:** Burp Suite Repeater
>
> **My approach:** I noticed the category filter was reflected in the request, so I intercepted it and tested how the application handled unexpected input.
> **Root cause:** user input was concatenated directly into the SQL query.
> **Fix:** parameterized queries, so input is always treated as data, never as SQL.
> **Lesson:** the bug is not the quote character. The bug is mixing data and code.

## 📊 Progress Tracker
### 📅 Study log

| Week | Focus | Labs | Links |
|---|---|:---:|---|
| 1 | SQL injection | 0 |  |


## 📚 Resources I Use

- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) and [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [TryHackMe](https://tryhackme.com/) · [OverTheWire](https://overthewire.org/wargames/)

## ⚖️ Ethics and Disclaimer

Everything in this repository comes from **intentionally vulnerable labs built for learning**.
I only test systems I own or have explicit permission to test. This content is for **educational purposes only**.
My write-ups are my own notes and my own words.

## 🤝 Connect With Me

I'd love to hear from fellow learners, mentors and security professionals. Feedback, corrections and advice are always welcome.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-connect-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/dina-jendeya-255207256)
[![GitHub](https://img.shields.io/badge/GitHub-follow-181717?logo=github&logoColor=white)](https://github.com/dinajendeya)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-profile-212C42?logo=tryhackme&logoColor=white)](https://tryhackme.com/p/dinajendeya)
[![Email](https://img.shields.io/badge/Email-contact-D14836?logo=gmail&logoColor=white)](mailto:deenayaseer@gmail.com)

Found a mistake in one of my write-ups? Please open an issue. Correcting me is a great way to help me learn.

---


*I'm still learning, and I'm sharing the journey as it happens. Thanks for stopping by, and feel free to follow along.* ⭐

