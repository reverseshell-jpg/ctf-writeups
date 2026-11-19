# DreamHack — Style Board

**Category:** Web Exploitation  
**Instance:** http://host3.dreamhack.games:19328/

---

## 1. Recon

**Global state**
- The global `users` dict stores the admin's username and password (`sha256(secret_key)`).

**`/login`**
- If credentials match an existing account in `users`, the server sets `session["is_admin"] = True`.
- Any other username (non-existent account) gets `session["is_admin"] = False`.
- **Logic flaw:** `is_admin` is set to `True` for *any* valid login, not exclusively for the `admin` account.

**`/mypage`**
- The rendered DOM contains the current user's `csrf-token` in a hidden input field.
- A `POST` request with a valid `csrf-token` and `session["is_admin"] == True` allows overwriting the admin's password. The target is hardcoded as `users["admin"]`, regardless of who sends the request.

**`/board/write`**
- Authenticated users can create posts.
- Post IDs are assigned sequentially (`len(posts) + 1`). A default post already exists at `id=1`, so attacker-created posts start from `id=2`.

**`/board/<int:post_id>`**
- Renders post content with **`autoescape` disabled**:
  ```jinja
  {% autoescape false %}
  {{ post.content }}
  {% endautoescape %}
  ```
- Raw HTML in `post.content` is injected directly into the DOM — no escaping of `<`, `>`, `"`, `'`, `&`.
- **Root cause of Stored XSS.**

**`/report`**
- Accepts a `path` parameter and uses a headless Chrome bot (logged in as `admin`) to visit `http://127.0.0.1:8000/{path}`.
- Bot lifetime is ~3 seconds — only same-origin requests complete in time; external HTTP callbacks do not (only the DNS query arrives before the bot is killed).

**`/style`**
- CSS preview page. User-supplied values (`font_family`, `bg_color`, etc.) are interpolated directly into a `<style>` block without server-side validation — **CSS Injection exists** here. Could be used to brute-force the CSRF token via attribute-selector leakage, but is unnecessary for this challenge.

---

## 2. Exploit

**Attack chain:**

```
       /login
          │
          ▼
Fetch own CSRF token from /mypage
          │
          ▼
POST /board/write with XSS payload as content
          │
          ▼
POST /report with path=board/2
          │   (admin bot visits the post)
          │   XSS executes in admin's browser session:
          │     1. fetch('/mypage') → extract admin's csrf-token from DOM
          │     2. POST /mypage with admin's token → overwrite admin password
          ▼
Login as admin with new password → GET /admin → Flag
```
## 3. Flag

<img width="1727" alt="exploit result" src="https://github.com/user-attachments/assets/cfb34e53-0d32-469f-bc22-cd957876a505" />
<img width="1364" alt="flag" src="https://github.com/user-attachments/assets/37884105-0345-4dbd-9651-d4e253ada2b0" />

`DH{**REDACTED**}`

---

*Additional note: CSS Injection exists at `/style` (unsanitized values interpolated into `<style>` block) and could theoretically be used to brute-force the admin CSRF token via attribute-selector side-channel. However, given that XSS already provides full same-origin script execution, CSS Injection is redundant for this challenge.*
