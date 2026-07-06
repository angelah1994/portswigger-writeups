# Blind SQL Injection with Conditional Errors

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
This write-up documents exploitation of a Blind SQL Injection with Conditional Errors vulnerability in a PortSwigger lab using Burp Suite against an Oracle database.

## Goal of the Lab
- Identify the vulnerability
- Identify Oracle DB
- Verify users table and administrator account
- Enumerate password length
- Extract password with Burp Intruder
- Log in as administrator

## Lab Information
- Platform: PortSwigger Web Security Academy
- Category: SQL Injection
- Difficulty: Practitioner
- Database: Oracle

## Tools Used
- Burp Suite Community
- Repeater
- Intruder
- Browser

## Vulnerability Overview
The application places the TrackingId cookie into a SQL query. Query results are hidden, but database errors are reflected in the HTTP response. By causing errors only when a condition evaluates to TRUE, information can be inferred one character at a time.

## Exploitation Steps

### Step 1 - Testing with a Single Quote
Payload:
```http
TrackingId=xyz'
```
Appending a single quote caused a database error, indicating unsanitized SQL input.

![Step1](images/01_Single_Quote_Error.png)

### Step 2 - Two Quotes
```http
TrackingId=xyz''
```
The error disappeared because the SQL syntax became balanced.

![Step2](images/02_Double_Quote_No_Error.png)

### Step 3 - Identify Oracle
```sql
TrackingId=xyz'||(SELECT '' FROM dual)||'
```
The request succeeded, confirming Oracle through the use of the DUAL table.

![Step3](images/03_Identify_Oracle_Database.png)

### Step 4 - Verify users Table
```sql
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM=1)||'
```
No error occurred, confirming the users table exists.

![Step4](images/04_Verify_Users_Table.png)

### Step 5 - Conditional Errors
TRUE payload generated an error using TO_CHAR(1/0); FALSE payload did not.

![True](images/05_Conditional_Error_True.png)
![False](images/06_Conditional_Error_False.png)

### Step 6 - Verify administrator
A CASE statement confirmed the administrator account exists by triggering an error only when the matching row was found.

![Admin](images/07_Verify_Administrator_User.png)

### Step 7 - Password Length
Repeated LENGTH(password)>N tests identified a 20-character password.

![Length](images/08_Password_Length_Enumeration.png)

### Step 8 - Burp Intruder
SUBSTR(password,position,1) was tested with characters a-z and 0-9 until the full password was recovered.

![Intruder](images/09_Password_Extraction_Intruder.png)

### Step 9 - Login
The recovered credentials were used to authenticate as administrator and solve the lab.

![Solved](images/10_Administrator_Login_Success.png)

## Result
The administrator password was successfully extracted and the lab solved.

## Skills Demonstrated
- Blind SQL Injection
- Oracle SQL
- Burp Repeater
- Burp Intruder
- Enumeration

## Mitigation
Use parameterized queries, validate input, suppress database errors, and apply least privilege.

## References
- PortSwigger Web Security Academy
- OWASP SQL Injection Prevention Cheat Sheet
