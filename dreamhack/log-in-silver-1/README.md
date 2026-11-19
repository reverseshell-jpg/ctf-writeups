# DreamHack Log-in Silver 1 — Web Exploit

## Key Functions

<img width="805" height="383" alt="image" src="https://github.com/user-attachments/assets/e154c3fd-19ca-41a3-ad01-faa18feb3d79" />

The `get_login_user` function scans `log.txt`, looking for lines that start with the string `"user="`, and returns the value from the **last matching line**.

<img width="1016" height="598" alt="image" src="https://github.com/user-attachments/assets/49546906-84eb-445c-bae1-78175df85287" />

The `calc_proxy` function calls `get_login_user` and checks whether the current user is `'admin'`. It then reads the `text` parameter and validates that it contains only characters from the set `0123456789+-*/`. If any other character is present, it returns `"Do not cheat!!!"`. Otherwise, the value is forwarded **as-is** to a PHP backend, and the PHP response is passed directly into `eval()`.

## Exploit

<img width="890" height="334" alt="image" src="https://github.com/user-attachments/assets/26abfeed-c5a6-48e9-a066-a71c1a6032f1" />

**Issue #1 — Log Injection.** The server writes debug logs directly to `log.txt` with no sanitization. This can be abused to inject a newline character and forge a new line reading `user=admin`, hijacking `get_login_user`'s trust in that file.

Take a look at [corydolphin/flask-cors](https://github.com/corydolphin/flask-cors) to inspect the library's source and see what gets logged server-side.

<img width="952" height="245" alt="image" src="https://github.com/user-attachments/assets/cd6ebd3e-0b58-4722-89d4-4055fcd922bc" />

The bug lives right here — this opens the door to HTTP header injection, or more precisely, **CRLF injection**.

> **Note:** Against older Werkzeug versions, attacking the `Origin` header directly used to work. On newer versions it's essentially blocked:
> ```python
> if request_origin:
>     LOG.debug("CORS request received with 'Origin' %s", request_origin)
> ```

For background on this class of bug, see HackTricks' [CRLF (%0D%0A) Injection](https://hacktricks.wiki/en/pentesting-web/crlf-0d-0a.html) page.

Time to run the attack:

> **Note:** Newer versions of Python's `requests` library block special/control characters as a mitigation, so it's best to use the low-level `socket` library instead if you're working in Python.

<img width="893" height="444" alt="image" src="https://github.com/user-attachments/assets/9cc60ee9-82b3-40c8-8274-de2d12a547b5" />

**Issue #2 — Bypassing the `calc_proxy` filter.** Looking at the flow, the `text` parameter is validated first and *then* forwarded to PHP — a textbook case for **Parameter Pollution**: when two parameters share the same name, Flask uses the **first** one while PHP uses the **second**.

Details: HackTricks' [Parameter Pollution](https://hacktricks.wiki/en/pentesting-web/parameter-pollution.html) page.

**Issue #3 — Python sandbox escape.** Finally, we need to bypass the restricted `eval()` sandbox (see HackTricks' [Bypass Python sandboxes](https://hacktricks.wiki/en/generic-methodologies-and-resources/python/bypass-python-sandboxes/index.html)):

- We can use a commonly-available class: `catch_warnings`.
- Since we don't know its exact index among subclasses, we build the full list and filter by name, then grab index `0`:
  ```python
  [c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == 'catch_warnings']
  ```
- Putting it all together to build the final payload and grab the flag:

  <img width="516" height="47" alt="image" src="https://github.com/user-attachments/assets/c0874bf8-6a30-48d9-90c8-12f692101005" />

**Done!**
