**SQL injection with filter bypass via XML encoding**

**Objective**

The objective of this lab was to exploit a SQL injection vulnerability in the product stock check feature. The application used XML requests to send product and store information. A Web Application Firewall (WAF) was blocking obvious SQL injection payloads, so the goal was to bypass the filter using XML encoding techniques and extract administrator credentials from the database.

**Vulnerability Identification**

**Analyzing the Stock Check Function**

The stock check feature sends a POST request containing XML data:

<stockCheck>
    <productId>1</productId>
    <storeId>1</storeId>
</stockCheck>

The storeId parameter was tested for SQL injection.

To determine whether user input was processed by the backend, mathematical expressions were submitted:

<storeId>1+1</storeId>

The application returned stock information for a different store, confirming that the input was being evaluated by the database query.

This indicated that the storeId parameter was vulnerable to SQL injection.

**Testing UNION SQL Injection**

The next step was determining the number of columns returned by the original SQL query.

A UNION-based payload was tested:

1 UNION SELECT NULL

Encoded inside the XML request:

<storeId>1 UNION SELECT NULL</storeId>
<img width="975" height="477" alt="image" src="https://github.com/user-attachments/assets/e288e06f-2cea-40d9-b606-cd5f6d6f8d3b" />

However, the request was blocked by the application's WAF.

The response indicated that the firewall detected SQL injection keywords such as:

UNION
SELECT
WAF Bypass Using XML Encoding

Because the application accepted XML input, the SQL payload could be obfuscated using XML entities.

The Hackvertor Burp Suite extension was used to encode the payload.

After encoding, the WAF no longer detected the malicious keywords.

The request was successfully processed, confirming the bypass.

Determining Column Count

Further testing showed that the SQL query returned only one column.

Attempts to return multiple columns caused an error response.

Therefore, the UNION query needed to return a single column.

Extracting User Credentials

Since only one column could be returned, the username and password fields needed to be combined.

The database contained a table: users with columns:username , password

The following SQL query was used:

1 UNION SELECT username || '~' || password FROM users

The payload was again encoded using Hackvertor before sending it through Burp Repeater.

Retrieving Administrator Credentials
<img width="975" height="549" alt="image" src="https://github.com/user-attachments/assets/720be836-8943-453b-9976-84b85d62b489" />


The response revealed the usernames and passwords stored in the database.

The administrator account credentials were identified:

**Tools Used**

**Tool	Purpose**

Burp Suite Repeater	Sending and modifying HTTP requests
Hackvertor Extension	Encoding SQL injection payloads
XML Entity Encoding	WAF bypass technique
PortSwigger Web Security Academy	Vulnerable testing environment

**Key Takeaways**
1. Input Validation Is Critical

Applications should never directly include user-controlled input in SQL queries.

Unsafe example:

SELECT * FROM stock WHERE storeId = 'USER_INPUT'

2. Parameterized Queries Prevent SQL Injection

Secure approach:

SELECT * FROM stock WHERE storeId = ?

The database treats user input as data instead of executable SQL.

3. WAFs Are Not Complete Security Solutions

A WAF can block common attack patterns, but attackers can bypass filters using:

Encoding
Obfuscation
Alternative syntax
Different representations of the same payload

Security must be implemented at the application level.

4. XML Parsing Can Introduce Security Risks

Applications processing XML should carefully validate and sanitize input before using it in database queries.

epth security controls.
