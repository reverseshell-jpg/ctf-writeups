# TryHackMe - Olympus

## 1. Reconnaissance

To access the website, add the target IP address to `/etc/hosts`.

<img width="1118" height="684" alt="image" src="https://github.com/user-attachments/assets/56b36302-dbf1-4820-8279-93be13bff66e" />

Use `ffuf` to perform a directory fuzzing attack:

<img width="778" height="186" alt="image" src="https://github.com/user-attachments/assets/46544e44-5288-445c-b04d-9c8815d1087c" />

## 2. Exploitation

### 2.1 Get shell

After observing and testing, I realized this website is vulnerable to SQL injection.

Although `sqlmap` could be used to attack the database, since this is a learning environment I prioritized performing the SQL injection manually.
(You can find out more in this video: https://youtu.be/Eg0jjrYr8GU?si=PDtMqwesHdUemAPa)

Next, I performed a UNION-based SQL injection attack:
- First, determine the number of columns.
- Next, list the table names and column names.
- Finally, extract the sensitive information.

**List the tables in the database:**

<img width="668" height="82" alt="image" src="https://github.com/user-attachments/assets/ed621e53-98e6-4770-93be-a48229715329" />

**Continue the SQL injection attack to retrieve the flag from the `flag` table:**

<img width="506" height="75" alt="image" src="https://github.com/user-attachments/assets/81e37fc7-e0b4-4345-bf59-17cbd4389b7f" />

**Retrieve the account information from the `users` table:**

<img width="811" height="192" alt="image" src="https://github.com/user-attachments/assets/c29cc42a-6df2-43e7-a558-0a54b087ee75" />

The users' passwords were encrypted using the Bcrypt algorithm; however, I was only able to crack the password of one user:

<img width="263" height="59" alt="image" src="https://github.com/user-attachments/assets/fdb721f7-6563-4623-8504-53ddc77d4cde" />

After logging into the **prometheus** account (a `user`-role account), I checked the CMS admin page for any vulnerabilities that could lead to an RCE. I didn't find any flaws there, but I noticed a very interesting detail instead:

<img width="131" height="71" alt="image" src="https://github.com/user-attachments/assets/64a588f8-1242-4df8-9e0e-8bb7cf196e63" />

Access `http://chat.olympus.thm/`:

<img width="397" height="387" alt="image" src="https://github.com/user-attachments/assets/7a1fa518-912c-440d-afce-0f77cacf7e8f" />

Log in with the prometheus account:

<img width="612" height="69" alt="image" src="https://github.com/user-attachments/assets/0a60719e-25bf-4a13-b8ca-6e299bf642cc" />

<img width="815" height="307" alt="image" src="https://github.com/user-attachments/assets/b5a96613-e18a-45d1-9b80-3df4c5cc18e4" />

Based on the two pieces of information above, the server allows file uploads and renames the uploaded files. We can access the uploaded files via the `/uploads/` API.

After some further research, I revisited a few database tables I hadn't read yet — and sure enough, they contained the information needed:

<img width="525" height="56" alt="image" src="https://github.com/user-attachments/assets/b6108d8e-5647-4341-9708-6560f8005f55" />

I uploaded a reverse shell PHP file, retrieved its new (renamed) filename via SQL injection, and then executed it:

<img width="1027" height="204" alt="image" src="https://github.com/user-attachments/assets/49942d5d-58ce-48ab-af14-fbe3db7eb935" />

The shell was successfully captured.

### 2.2 Log in via SSH as user `zeus`

Access zeus's home directory and read the second flag:

<img width="422" height="37" alt="image" src="https://github.com/user-attachments/assets/6c141db3-ec7d-41df-a36f-3303e6b7dfd5" />

I also tried accessing the `.ssh` folder to find a private key, but didn't have sufficient permissions.

Next, I enumerated SUID binaries and found an interesting one:

<img width="171" height="21" alt="image" src="https://github.com/user-attachments/assets/0f92a4b6-02d1-419e-b15f-17a5d70fbe87" />

Using `strings` to analyze the SUID binary, I found that this file is used to copy other files. So I tried using it to read/copy `id_rsa`:

<img width="626" height="226" alt="image" src="https://github.com/user-attachments/assets/c8e101cb-88cc-4b96-9222-1f0379fe7c79" />

I successfully obtained the private key. Although it is passphrase-protected, that passphrase can be cracked offline.

### 2.3 Privilege Escalation

After using LinPEAS and reading `/etc/crontab` to list files that `zeus` can run, I found an interesting PHP file:

<img width="529" height="21" alt="image" src="https://github.com/user-attachments/assets/6ade9f9d-6251-423d-84c0-5754e26f3c11" />

This turned out to be a built-in "backdoor" — not a real web exploit, but more of an Easter-egg backdoor baked into the machine. The key point: the PHP script simply executes a separate privileged binary to gain root; the binary itself is what grants root, and PHP is just one way to trigger it over the web. Since we already have SSH access as `zeus`, there's no need to go through the web at all — we can just run the binary directly:

<img width="1742" height="488" alt="image" src="https://github.com/user-attachments/assets/351e6693-2c0f-435e-a1db-4455cdcc28fc" />

<img width="796" height="98" alt="image" src="https://github.com/user-attachments/assets/84cfd160-45eb-4582-889f-b7fd056be939" />

**Flag #3:**

<img width="393" height="146" alt="image" src="https://github.com/user-attachments/assets/3ce4fe59-861b-4de6-88b2-2e11c4d14480" />

**Flag #4** (found using `grep` to search for the remaining flag):

<img width="393" height="146" alt="image" src="https://github.com/user-attachments/assets/692427e8-ed85-4933-98df-370c3db38932" />
