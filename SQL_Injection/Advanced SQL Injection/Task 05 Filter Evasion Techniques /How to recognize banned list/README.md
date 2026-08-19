The easiest way is to **look at the application's filtering code or test one character/keyword at a time**.

Example:

```php
$special_chars = array(" ", "AND", "and", "or", "OR", "UNION", "SELECT");
$username = str_replace($special_chars, '', $username);
```

So here **banned list**:

```text
BANNED
───────
Space " "
AND
and
or
OR
UNION
SELECT
```

### How to recognize it in a real?

#### 1. Look for filtering code

Search the source/code for things like:

```text
str_replace()
preg_replace()
filter_var()
blacklist
sanitize
WAF rules
```

For example:

```php
str_replace("OR", "", $input);
```

Immediately tells you:

> `OR` is being removed.

---

#### 2. Compare the input with the generated SQL

Example:

```text
Input:
1' OR 1=1

        ↓ filter

Generated SQL:
1'  1=1
```

You can see that `OR` disappeared.

Therefore:

```text
OR → BANNED/FILTERED
```

This is one of the most useful techniques.

---

#### 3. Test individually

Don't start with a complicated payload.

Try:

```text
OR
AND
UNION
SELECT
```

and observe what the application does.

For example:

```text
Input:       1 OR 1=1
After filter: 1  1=1
```

Then you know:

```text
OR → filtered
```

---

### 4. Test spaces separately

If you suspect spaces are filtered:

```text
Input:
1 2 3
```

If the application processes it as:

```text
123
```

then:

```text
SPACE → filtered
```

That's why you can use:

```text
%0A
%09
```

or comments instead of ordinary spaces.

---

### 5. Important: "banned" doesn't always mean completely blocked

There are two common situations:

**Blocked:**

```text
OR → request rejected
```

**Removed:**

```text
1 OR 1=1
    ↓
1  1=1
```


### 🧠 Pentesting thought process

Always think:

```text
1. What input is accepted?
          ↓
2. What gets removed/changed?
          ↓
3. What characters/keywords are filtered?
          ↓
4. What SQL syntax is still available?
          ↓
5. Can the intended SQL meaning be represented another way?
```

