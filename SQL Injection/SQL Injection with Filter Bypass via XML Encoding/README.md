# SQL Injection with Filter Bypass via XML Encoding

## Lab Overview

This PortSwigger Web Security Academy lab demonstrates a SQL injection vulnerability in the product stock check functionality.

The application uses XML requests to communicate with the backend server. A Web Application Firewall (WAF) is implemented to block common SQL injection payloads. However, the filtering mechanism can be bypassed by encoding the SQL payload using XML entities.

The vulnerability allows an attacker to perform a UNION-based SQL injection attack, extract sensitive information from the database, and retrieve administrator credentials.


## Lab Objective / Goal

The goal of this lab is to:

- Identify the SQL injection vulnerability in the stock check feature.
- Determine how the backend SQL query processes user input.
- Bypass the WAF protection using XML encoding.
- Perform a UNION SQL injection attack.
- Extract administrator credentials from the database.
- Log in as the administrator user.

---

## Vulnerability Description

The vulnerability exists in the `storeId` parameter of the stock check XML request.

The application directly incorporates user-controlled input into an SQL query without proper validation or parameterization.

Because the application returns database query results in the response, a UNION-based SQL injection attack can be used to retrieve information from other database tables.

---

## Lab Environment

**Platform:** PortSwigger Web Security Academy  
**Difficulty:** Practitioner  
**Vulnerability Type:** SQL Injection  
**Attack Type:** UNION-based SQL Injection  
**Injection Point:** `storeId` XML parameter

---

## Tools Used

| Tool | Purpose |
|---|---|
| Burp Suite Community Edition | Intercepting and modifying HTTP requests |
| Burp Repeater | Testing SQL injection payloads |
| Hackvertor Extension | Encoding SQL payloads to bypass WAF |
| Browser | Accessing the vulnerable application |

---

# Exploitation Steps

## Step 1: Intercepting the Stock Check Request

The stock check feature sends an XML request:

```xml
<stockCheck>
    <productId>1</productId>
    <storeId>1</storeId>
</stockCheck>
```

The `storeId` parameter was identified as a potential injection point because it is controlled by user input.

---

## Step 2: Testing for SQL Injection

A mathematical expression was inserted into the parameter:

```xml
<storeId>1+1</storeId>
```

The application returned a different stock value, confirming that the input was being processed by the backend.

This indicated that the parameter was vulnerable to SQL injection.

---

## Step 3: Testing UNION Injection

A UNION query was attempted to identify the number of columns returned by the original SQL query:

<img width="975" height="477" alt="image" src="https://github.com/user-attachments/assets/4b789f13-a062-4738-a261-54d4c5849af4" />


The request was blocked by the Web Application Firewall (WAF).

The WAF detected SQL keywords such as:

- UNION
- SELECT

---

## Step 4: Bypassing the WAF Using Hackvertor

Since the application processes XML data, XML entity encoding was used to obfuscate the SQL payload.

Using Burp Suite Hackvertor:

```
Extensions → Hackvertor → Encode → hex_entities
```

The SQL injection payload was converted into XML encoded characters.

Example:

Original payload:

```sql
UNION SELECT NULL
```

The WAF failed to recognize the malicious SQL keywords, allowing the request to execute.

---

## Step 5: Determining the Number of Columns

Testing showed that the original SQL query returned only one column.

Attempts to return multiple columns caused an error response.

Therefore, the UNION query was modified to return a single column.

---

## Step 6: Extracting User Credentials

The database contained a `users` table with:

- username
- password

Because only one column could be returned, both values were concatenated:

<img width="975" height="549" alt="image" src="https://github.com/user-attachments/assets/8ae8da7f-c016-4c1e-b7c3-430733dc98db" />


The response returned stored credentials:


# Remediation

To prevent SQL injection vulnerabilities:

## 1. Use Parameterized Queries

Avoid directly inserting user input into SQL queries.

Unsafe:

```sql
SELECT * FROM stock WHERE storeId = 'input'
```

Secure:

```sql
SELECT * FROM stock WHERE storeId = ?
```

---

## 2. Validate User Input

Implement strict validation rules for all user-controlled data.

---

## 3. Apply Least Privilege

Database accounts used by applications should have only the permissions they require.

---

## 4. Do Not Rely Only on WAF Protection

WAFs can help detect attacks but can often be bypassed through:

- Encoding
- Obfuscation
- Alternative payload formats

Security controls should be implemented within the application itself.

---

# Key Takeaways

This lab demonstrated:

- How SQL injection vulnerabilities can exist in XML-based applications.
- How UNION SQL injection can extract database information.
- How attackers bypass WAF protections using encoding techniques.
- The importance of secure database interaction using prepared statements.

---

## References

- PortSwigger Web Security Academy - SQL Injection Labs
- OWASP SQL Injection Prevention Cheat Sheet
