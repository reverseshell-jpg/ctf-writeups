# Ollie - TryHackMe Writeup

## 1. Reconnaissance

<img width="420" height="93" alt="image" src="https://github.com/user-attachments/assets/bf376bc8-6137-4a45-a95b-f282a36fa675" />

The server has 3 open ports.

**Port 80 is open:**

<img width="598" height="491" alt="image" src="https://github.com/user-attachments/assets/e2ae63aa-02d7-4f78-a721-6c5e2bde269f" />

Tried SQL injection and common username/password combos such as `admin/admin`, `root/root`, etc., but none worked. Set this aside for now.

**Back to port 1337:**

Connected to port 1337 using `nc`. After answering a few simple questions, obtained a set of credentials:

<img width="1241" height="155" alt="image" src="https://github.com/user-attachments/assets/18b6ecc9-0c32-44f2-90dd-b57d2180471d" />

## 2. Exploitation

### 2.1 Getting a Shell

After logging in and poking around, discovered the server is running **phpIPAM**, an outdated version — **1.4.5**.

<img width="409" height="138" alt="image" src="https://github.com/user-attachments/assets/7783f93c-3acd-4f6b-bcd9-ba1834f5d7e1" />

Research turned up **CVE-2022-23046**, which can be used to get a shell:

- The vulnerability is a SQL injection in the `app/admin/routing/edit-bgp-mapping-search.php` endpoint, which can be abused with MySQL's `INTO OUTFILE` to write an executable file to disk.

PoCs for CVE-2022-23046 are available on GitHub:
- https://github.com/dnr6419/CVE-2022-23046
- https://github.com/behindsecurity/phpipam-rce-authenticated

Alternatively, the file can be uploaded manually using `sqlmap`:

- Navigate to **Routing**, open a peer, click **Action → Subnet mapping**, and perform any search in the **search subnet** field (be sure to intercept the request with Burp Suite while searching).
- Copy the intercepted request from Burp Suite into a `.txt` file, prepare a reverse-shell payload file, and use `sqlmap` to upload it to the server.

<img width="1286" height="214" alt="image" src="https://github.com/user-attachments/assets/c2baf027-7e01-464f-ba54-db7a89656590" />

- Navigate to `/reverse.php` — shell obtained successfully:

<img width="1037" height="179" alt="image" src="https://github.com/user-attachments/assets/b9c5687f-0cee-4134-848c-3da67a5b5786" />

### 2.2 Privilege Escalation

#### 2.2.1 Pivoting to user `ollie`

- After browsing to `/home`, noticed two user directories: `ollie` and `ubuntu`.
- Tried switching to the `ollie` user using the password obtained earlier from port 1337 — it worked.

<img width="305" height="41" alt="image" src="https://github.com/user-attachments/assets/c6b3ac90-57e6-4f16-b916-e2cd6213f732" />

#### 2.2.2 Escalating to root

- After downloading and running `pspy64`, noticed root regularly runs two programs: `olliebot.py` and `feedme`.

<img width="600" height="20" alt="image" src="https://github.com/user-attachments/assets/94af61e6-1afb-4a4c-9f5b-9087ba9d6e92" />

<img width="652" height="68" alt="image" src="https://github.com/user-attachments/assets/aa8f303b-6cb2-4141-9c58-25bff502a990" />

- Used `find` to locate these two programs, but could only locate `feedme`.

<img width="569" height="60" alt="image" src="https://github.com/user-attachments/assets/9441edf9-7101-4c42-a1f1-a407e5baeaa6" />

- Checked `/usr/bin/feedme` and found write permission on the file, so added the SUID bit to `/bin/bash` to spawn a root shell.

<img width="597" height="155" alt="image" src="https://github.com/user-attachments/assets/6af6d810-e42f-4af9-8495-f08b58a3e50f" />

- Finally, grabbed the flag:

<img width="991" height="81" alt="image" src="https://github.com/user-attachments/assets/0e892a16-2e91-43e4-8e33-78103c2ec228" />

**Done!**
