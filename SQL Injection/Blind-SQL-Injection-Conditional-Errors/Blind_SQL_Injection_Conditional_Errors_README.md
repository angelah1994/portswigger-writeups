# Blind SQL Injection with Conditional Errors (PortSwigger)

## Table of Contents
- Overview
- Goal of the Lab
- Lab Information
- Tools Used
- Vulnerability Overview
- Exploitation Steps
- Result
- Skills Demonstrated
- Mitigation
- References

## Overview
This lab demonstrates exploiting a blind SQL injection vulnerability that reveals information through conditional database errors. The injection point is the `TrackingId` cookie. By triggering Oracle database errors only when specific conditions evaluate to true, it is possible to enumerate the administrator's password and authenticate as that user.

## Goal of the Lab
Recover the administrator password using blind SQL injection with conditional errors and log in as the administrator.

## Lab Information
- Platform: PortSwigger Web Security Academy
- Difficulty: Practitioner
- Database: Oracle
- Injection Point: TrackingId cookie

## Tools Used
- Burp Suite Community Edition
- Burp Repeater
- Burp Intruder
- Web Browser

## Vulnerability Overview
The application concatenates the TrackingId cookie into a SQL query without proper sanitization. Query results are not displayed, but database errors are observable, enabling boolean inference through conditional errors.

## Exploitation Steps
1. Intercept the request and identify the `TrackingId` cookie.
2. Append a single quote (`TrackingId=xyz'`) and observe an error.
3. Append two quotes (`TrackingId=xyz''`) and observe the error disappear.
4. Confirm SQL execution with `SELECT '' FROM dual`.
5. Verify the Oracle database using the `dual` table.
6. Verify the `users` table exists.
7. Trigger conditional errors using `CASE WHEN`.
8. Confirm the `administrator` account exists.
9. Determine the administrator password length.
10. Use Burp Intruder to enumerate each password character.
11. Log in using the recovered password.

## Result
Successfully recovered the administrator password and authenticated as the administrator, completing the lab.

## Skills Demonstrated
- Blind SQL Injection
- Oracle SQL syntax
- Burp Repeater
- Burp Intruder
- Error-based inference
- Authentication bypass through credential extraction

## Mitigation
- Use parameterized queries.
- Validate and sanitize user input.
- Avoid exposing database errors.
- Apply least privilege to database accounts.
- Monitor and log SQL injection attempts.

## References
- PortSwigger Web Security Academy – Blind SQL Injection with Conditional Errors
- OWASP SQL Injection Prevention Cheat Sheet
