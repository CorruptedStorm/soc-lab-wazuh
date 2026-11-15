# Home SOC Lab with Wazuh

## What this project is

I built a small, isolated network of virtual machines to practice two things: setting up a security monitoring tool (Wazuh), and testing whether it actually notices when something happens.

The setup: one VM acts as the SIEM (Wazuh), watching two other VMs (one Ubuntu, one Windows) that stand in for computers on a network. A fourth VM plays as an attacker. I ran a couple of common, well-known attacks against the victim VMs from Kali, then checked whether Wazuh's dashboard actually caught them.

All of this ran on a private, isolated network I built myself (192.168.56.0/24).

**Tools used:** Wazuh (the monitoring tool), VirtualBox (to run the VMs), Kali Linux (attack tools), Hydra (password guessing tool), Nikto (website scanner)

---

## The setup

| VM | What it's for | IP address |
|---|---|---|
| Wazuh server | Watches the other machines and shows alerts | 192.168.56.10 |
| Ubuntu victim | Stand-in target machine/server | 192.168.56.11 |
| Windows victim | Stand-in target machine/host | 192.168.56.12 |
| Kali | Attacker machine | 192.168.56.13 |

**Wazuh server running:**
<img width="1101" height="1024" alt="image" src="https://github.com/user-attachments/assets/833efc5a-505e-4cb6-b0c8-587221e82ad0" />

**Windows VM (victim):**
<img width="1097" height="1073" alt="image" src="https://github.com/user-attachments/assets/8e84af00-8764-422e-b9d3-6985d2c96988" />

**Ubuntu VM (victim):**
<img width="1101" height="1030" alt="image" src="https://github.com/user-attachments/assets/612ab718-555d-4fbe-a5bc-76eab15ba07e" />

**Kali VM (attacker):**
<img width="1091" height="1053" alt="image" src="https://github.com/user-attachments/assets/66c23bc1-e5cc-4266-b6c1-2da6908a7aed" />

Before running any attacks, I confirmed both victim machines were reporting in and showing as active:

<img width="2550" height="783" alt="image" src="https://github.com/user-attachments/assets/d30e0b6e-caf1-4e02-8e90-b3655bf2b388" />

---

## Test 1: Guessing a password (brute force)

**What I did:** I created a test account on the Ubuntu VM with a weak password, then used a tool called Hydra to automatically try a short list of common passwords against it over SSH.

```
┌──(user㉿kali)-[~]
└─$ hydra -l labuser -P pw.txt ssh://192.168.56.11 -t 4
Hydra starting at 2026-11-08 14:10:07
[DATA] attacking ssh://192.168.56.11:22/
[22][ssh] host: 192.168.56.11   login: labuser   password: sunshine1
1 of 1 target successfully completed, 1 valid password found
Hydra finished at 2026-11-08 14:10:14
```

**What Wazuh caught:** Wazuh flagged it. The alert description literally read **"Multiple authentication failures followed by a success"**. This was one of the more serious alerts it raised (severity 12 out of 15).

<img width="1919" height="954" alt="image" src="https://github.com/user-attachments/assets/f7b0dc7e-8523-4d02-9dda-ebf1068ed408" />

**Takeaway:** This confirms Wazuh can detect brute-force login attempts, and it labels them correctly.

---

## Test 2: Scanning a website for weaknesses

**What I did:** I ran a tool called Nikto against the website running on the Ubuntu VM. Nikto automatically checks a web server for a long list of common misconfigurations and known weaknesses.

```
┌──(user㉿kali)-[~]
└─$ nikto -h http://192.168.56.11
- Nikto v2.6.1
+ Target IP:          192.168.56.11
+ Server: Apache/2.4.66 (Ubuntu)
+ Apache/2.4.66 appears to be outdated (current is at least 2.4.68).
+ Suggested security headers missing (several)
+ 8107 requests: 0 errors and 9 items reported on the remote host
+ End Time: 2026-11-08 14:16:23 (GMT-5) (16 seconds)
```

**What Wazuh caught:** This scan generated a lot more noise (over 7,900 total log entries, since Nikto makes thousands of requests), but a few stood out as serious:

- Wazuh flagged that the scan was specifically testing for a well-known, named security bug (it labeled it as a "Shellshock" attack attempt. After looking into it more, this only means Wazuh noticed the *attempt*, not that the bug was actually present on my server.
- It also raised one alert about a user account being added around the same time, which doesn't match anything Nikto normally does. I haven't confirmed yet whether that's connected to the scan or just a coincidence from something else I did around that time — worth double-checking.

<img width="1920" height="953" alt="image" src="https://github.com/user-attachments/assets/45f90711-6235-4187-9bba-b041d2122b9e" />

**Takeaway:** Wazuh picked up on the scan testing for a specific known vulnerability, even though my server probably isn't actually vulnerable to it.An alert firing means "a known attack pattern was seen," not necessarily "the attack succeeded."

---

## What I learned

- How to set up a monitoring tool and get data on attacks.
- A simple, well-known attack (password guessing) produced a clear, easy-to-read alert.
- A broader scan produced a lot of alerts, and part of the skill is figuring out which ones actually matter versus which are just noise.
- Not every alert that looks alarming (e.g., "attack followed by user creation") means what it sounds like — it's worth checking before assuming the worst.

## What I'd like to try next
- Test whether a plain network scan (just checking which doors/ports are open, without attacking anything) gets noticed at all, and if not, why.
- Look into the "user added" alert above and figure out what actually caused it.
- Try writing one simple custom alert rule of my own.
