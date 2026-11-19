# dream_hack-admin-only
1. Recon
   After using dirsearch to scan:
   <img width="768" height="290" alt="image" src="https://github.com/user-attachments/assets/64764519-0d0c-498d-b6ca-3b577ffb00c4" />

   - Access the /robots.txt endpoint:
     <img width="369" height="75" alt="image" src="https://github.com/user-attachments/assets/23aa1b03-5063-4544-9701-88fc877cac5b" />
     
   - See the /getflag endpoint and try accessing it:
     <img width="282" height="74" alt="image" src="https://github.com/user-attachments/assets/acb0e23d-c0c0-4297-9743-bd43682ac53f" />
     
   - Can't access, temporarily skip and return to another endpoint, /search:
     <img width="715" height="280" alt="image" src="https://github.com/user-attachments/assets/cafa16cf-63f9-402a-8e71-22e7379bbd37" />
2. Exploit
   - Let's try sql injection - (Union select) to see if we can get the admin account:
     + First we need to determine which database the server uses.
     + Next, we need to determine the number of columns in the database table: Use ORDER BY
     + Next, you need to determine the table name and field name: Use table schema sqlite3
       <img width="1247" height="363" alt="image" src="https://github.com/user-attachments/assets/b85888cf-4c68-43cc-a5e4-2c74d586c593" />
   - After logging in with the admin account and observing, I realized the mining operation was located here:

       <img width="1166" height="591" alt="image" src="https://github.com/user-attachments/assets/1d460cd0-22aa-4359-8d0f-6743c96f3950" />
   - Next, let's try entering http://127.0.0.1:5000 to see if it works. If it does, then we can be sure this is an SSRF error.
     Note: The reason for opening port 5000 on localhost is because the instance points 17332/tcp → 5000/tcp.
       <img width="627" height="142" alt="image" src="https://github.com/user-attachments/assets/36383136-ca99-4235-a0c0-8485b2edf145" />
   - It has been confirmed that it is ssrf, we tried to use that to access /getflag, but it seems to be blocked by the WAF:
       <img width="604" height="136" alt="image" src="https://github.com/user-attachments/assets/9ca6ad2a-f9e4-4049-9cff-b0eb90d695ed" />
   - After some research, I think it's possible to use URL encoding to bypass WAF, but typical websites only encode special characters, so we have to manually encode them using a script:
       <img width="1238" height="217" alt="image" src="https://github.com/user-attachments/assets/c36c8fda-2dcc-445a-af20-6333e6e5d8a2" />
   - After re-inserting the payload and re-checking the avatar URL, we successfully obtained the flag:
       <img width="680" height="463" alt="image" src="https://github.com/user-attachments/assets/be4d6df9-2211-47ca-b73d-54b1ea3336c1" />





     
