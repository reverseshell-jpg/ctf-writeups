THM-Athen 
IP INSTANCE: 10.48.150.98
1. Recon
use nmap scan:
<img width="1224" height="525" alt="Screenshot 2026-09-20 171425" src="https://github.com/user-attachments/assets/c6310712-b7e5-4fc4-84ab-31d56b0be72d" />
Connect smb server with anonymous account:
<img width="1217" height="216" alt="Screenshot 2026-09-20 171540" src="https://github.com/user-attachments/assets/cc02b214-958d-4785-b8e5-f7d742d001b6" />
Access public resources and read the txt file:
<img width="1649" height="186" alt="Screenshot 2026-09-20 171727" src="https://github.com/user-attachments/assets/c5f0b804-7e10-4e8e-903b-16887823274f" />
It's easy to see there's a 'ping ip' program in /myrouterpanel.
2. Exploit:
access the link http://10.48.150.98/myrouterpanel/ 
<img width="1872" height="538" alt="Screenshot 2026-09-20 171948" src="https://github.com/user-attachments/assets/ae986a40-3800-4416-adc6-6efab1bf87c4" />
It's easy to recognize this as a form of command injection: https://www.hackerone.com/blog/how-command-injections
After a few trials, I realized the website was filtering out some special characters like: ; | & .vv. Finally, I figured out that ` was usable.
Create reverse shell: https://www.revshells.com/
<img width="585" height="117" alt="Screenshot 2026-09-20 173335" src="https://github.com/user-attachments/assets/c5561c1b-8f0c-4db8-9d3d-1c6d821df48c" />
After searching through the suid files using linpes and pspy64, I discovered something interesting:
<img width="999" height="102" alt="Screenshot 2026-09-20 174707" src="https://github.com/user-attachments/assets/4919248c-9bb4-4106-959e-a5f6d3958feb" />
Athena executed the file /usr/share/backup/backup.sh. I checked /usr/share/backup/backup.sh and realized I had permissions to edit it, so I decided to change the backup.sh file to reverse shell.
<img width="869" height="203" alt="Screenshot 2026-09-20 175229" src="https://github.com/user-attachments/assets/d0d4696d-4959-486b-a97f-5ef355a66a43" />
Then we can get user.txt:
<img width="348" height="34" alt="Screenshot 2026-09-20 182414" src="https://github.com/user-attachments/assets/efe3bf66-7e25-47ff-9f31-1fcd72634fd7" />
4. Privilege Escalation:
<img width="819" height="136" alt="Screenshot 2026-09-20 175408" src="https://github.com/user-attachments/assets/01a8a8e9-817a-4f80-86fa-d42020624cc6" />
As you can see, we can use the root role to load the model.
Use strings to see if any information can be extracted:
<img width="899" height="460" alt="Screenshot 2026-09-20 182617" src="https://github.com/user-attachments/assets/1f347582-d089-4ced-9253-934f32384874" />
This appears to be m0nad's LLM rootkit model: https://github.com/m0nad/Diamorphine 
After carefully reading the GitHub link above, I tried following the instructions using `kill -64 0`, but it didn't root my device. After a while, I think the ROM creator intentionally modified the signal to allow rooting.
Therefore, I decided to download venom.ko and decompile it.
After using Ghidra to reverse engineer it, I discovered a special function called hacked_kill:
<img width="993" height="304" alt="Screenshot 2026-09-20 183626" src="https://github.com/user-attachments/assets/e57bb184-18d0-49db-907c-754e7d9cc75c" />
And the content of the give_root function is:
<img width="1867" height="440" alt="Screenshot 2026-09-20 183402" src="https://github.com/user-attachments/assets/4b1efd05-4451-46c5-b4c6-14a5a59acc1f" />
In short, we will pass in a special number, and if that number is correct, we will call the give_root function and overwrite all uid/gid with 0(root), and 0x39 convert to int is: 57
Therefore, we can access root after importing the model using the command `kill -57 0`.
<img width="815" height="119" alt="Screenshot 2026-09-20 184602" src="https://github.com/user-attachments/assets/223dbb19-ce36-42b9-b6e1-9163d9ca67b7" />
Get root.txt:
<img width="771" height="32" alt="Screenshot 2026-09-20 184643" src="https://github.com/user-attachments/assets/ab3928c4-c6c2-4be5-ab35-23f09d5f200c" />


