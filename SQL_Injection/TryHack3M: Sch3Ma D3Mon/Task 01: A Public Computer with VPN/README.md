

## 1. Objective

The objectives are:

1. Decrypt the captured HTTPS/TLS traffic.
2. Find the HTTP request where the suspect logs into the criminal marketplace.
3. Extract:
    - **Username**
    - **Password**

The key idea is:

```
Encrypted PCAP
      +
TLS key log
      ↓
Wireshark decrypts TLS
      ↓
HTTP/HTTP2 traffic becomes visible
      ↓
Find login request
      ↓
Username + Password
```

---

# 2. Locate the Task Files

Download task file or for AttackBox look into:

```bash
/root/Rooms/TryHack3M/sch3MaD3Mon
```

Open a terminal and run:

```bash
cd /root/Rooms/TryHack3M/sch3MaD3Mon
```

Then:

```bash
ls -lah
```

You should see the capture file and the TLS key-log file.

---

# 3. Open the PCAP in Wireshark

Run:

```bash
wireshark
```

Then:

**File → Open**

Navigate to:

```
/root/Rooms/TryHack3M/sch3MaD3Mon/
```

Select the `.pcap` or `.pcapng` file.

At this point, the traffic will probably still appear as encrypted TLS traffic.

---

# 4. Load the TLS Key Log

This is the most important step.

In Wireshark:

```
Edit
  ↓
Preferences
  ↓
Protocols
  ↓
TLS
```

Find:

```
(Pre)-Master-Secret log filename
```

Click:

```
Browse...
```

Select:

```
/root/Rooms/TryHack3M/sch3MaD3Mon/ssl-key.log
```

Then click:

```
OK
```


<img width="1278" height="713" alt="Screenshot 2026-08-29 at 2 14 17 PM" src="https://github.com/user-attachments/assets/7ad7a903-9694-4dc9-932d-94be5a8be6b5" />

---

# 5. Confirm That Decryption Worked

In the Wireshark display-filter box, try:

```
http
```

Press **Enter**.

If the traffic uses HTTP/2, also try:

```
http2
```

You can also try:

```
tls
```

before and after importing the key file.

After successful decryption, you should start seeing higher-level application information instead of only encrypted TLS application data.

---

# 6. Find the Login Request

Now the goal is to locate the request where the suspect submits credentials.

Try:

```
http.request
```

If the site uses HTTP/2:

```
http2
```

Look through the decrypted requests for things such as:

```
POST
/login
/login.php
/auth
/signin
```

You can also search for likely credential-related strings.

Use:

```
http.request.method == "POST"
```

This is particularly useful because login credentials are commonly submitted through POST requests.




<img width="1279" height="740" alt="Screenshot 2026-08-29 at 2 15 44 PM" src="https://github.com/user-attachments/assets/c0cb92c8-8fcb-442c-bd08-f74b74e8663e" />

---

# 7. Inspect a Suspicious Packet

Look for Post request like below:

```
78	30.606734195	10.11.81.126	10.10.111.136	HTTP	1009	POST /login.php?msg=1 HTTP/1.1  (application/x-www-form-urlencoded)
```

or:

```
Hypertext Transfer Protocol 2
```

Then look for:

```
Request Method: POST
```

and the request data.

For HTTP/2, the credentials may appear in the reconstructed/decrypted request data rather than in the traditional HTTP/1.1 layout.

---

# 8. Look for Username and Password

Click on the request and look for below:

```
Frame 78: 1009 bytes on wire (8072 bits), 1009 bytes captured
Linux cooked capture v1
Internet Protocol Version 4, Src: 10.11.81.126, Dst: 10.10.111.136
Transmission Control Protocol, Src Port: 41908, Dst Port: 8443
Transport Layer Security
Hypertext Transfer Protocol
HTML Form URL Encoded: application/x-www-form-urlencoded
    Form item: "uid" = "lannister"
    Form item: "password" = "hrpTfL42wMv3"
```


<img width="1278" height="732" alt="Screenshot 2026-08-29 at 2 16 29 PM" src="https://github.com/user-attachments/assets/587929a4-ce5f-4838-a9b9-ee79e395dd29" />




Or do following:

```
Right Click
  ↓
Follow
  ↓
HTTP Stream
```

<img width="1278" height="742" alt="Screenshot 2026-08-29 at 2 16 52 PM" src="https://github.com/user-attachments/assets/83e574b5-6d75-4041-9d32-e9f6a4203f34" />



You will find something like below:

```
uid=lannister&password=hrpTfL42wMv3
```



<img width="1281" height="747" alt="Screenshot 2026-08-29 at 2 17 21 PM" src="https://github.com/user-attachments/assets/450d0957-9c18-4cc3-99c9-9bbbc2e7801c" />



---

# 

---

# 16. Extract the Two Answers

Once you find the login request, you should see something like:

```
POST /login.php?msg=1 HTTP/1.1
Host: 10.10.111.136:8443
Connection: keep-alive
Content-Length: 35
Cache-Control: max-age=0
sec-ch-ua: "Not(A:Brand";v="24", "Chromium";v="122"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Linux"
Upgrade-Insecure-Requests: 1
Origin: https://10.10.111.136:8443
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://10.10.111.136:8443/login.php?msg=1
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=5ebdc3db7dbc103b5f56ed1e2bcd6610

uid=lannister&password=hrpTfL42wMv3
```

Then:

### Question 1

Copy the value after:

```
username=lannister
```

Submit that as:

```
Suspect's username
```

### Question 2

Copy the value after:

```
password=hrpTfL42wMv3
```

Submit that as:

```
Suspect's password17. Recommended Wireshark Workflow
```

---

