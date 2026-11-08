# SOC Lab: Wazuh SIEM Detection & Attack Simulation

A home lab built to practice deploying and operating a SIEM. Wazuh monitors two victim machines (Linux and Windows) while Kali Linux is used to simulate common attacks, validate what Wazuh detects, and identify detection gaps. All testing was performed against self-owned VMs on an isolated host-only network (192.168.56.0/24) — no production or third-party systems were targeted.

**Tools used:** Wazuh (SIEM/XDR), VirtualBox, Kali Linux, Hydra, Nikto, Sysmon <!-- add Suricata, nmap, etc. as you add them -->

---

## Architecture

| VM | Role | OS | Host-only IP |
|---|---|---|---|
| Wazuh server | Manager, Indexer, Dashboard | Ubuntu 24.04 | 192.168.56.10 |
| Ubuntu victim | Endpoint (SSH + Apache) | Ubuntu 26.04 LTS | 192.168.56.11 |
| Windows victim | Endpoint | Windows 11 Home | 192.168.56.12 |
| Kali | Attacker | Kali 2026 | 192.168.56.13 |

<!-- Optional: add a simple network diagram image here -->

**Wazuh server running aggregation:**
<img width="1101" height="1024" alt="image" src="https://github.com/user-attachments/assets/833efc5a-505e-4cb6-b0c8-587221e82ad0" />

**Windows 11 VM (victim):**
<img width="1097" height="1073" alt="image" src="https://github.com/user-attachments/assets/8e84af00-8764-422e-b9d3-6985d2c96988" />

**Ubuntu VM (victim):**
<img width="1101" height="1030" alt="image" src="https://github.com/user-attachments/assets/612ab718-555d-4fbe-a5bc-76eab15ba07e" />

**Kali VM (attacker):**
<img width="1091" height="1053" alt="image" src="https://github.com/user-attachments/assets/66c23bc1-e5cc-4266-b6c1-2da6908a7aed" />

---

## Setup

- Deployed Wazuh agents on both victim VMs; confirmed both report as **Active** in the dashboard before any testing.
- <!-- Add: FIM directories monitored, Apache log forwarding, Sysmon config used (e.g. SwiftOnSecurity), any other log sources -->

**Wazuh dashboard before attacks — both agents active:**
<img width="2550" height="783" alt="image" src="https://github.com/user-attachments/assets/d30e0b6e-caf1-4e02-8e90-b3655bf2b388" />

---

## Attack Simulations

### 1. Port scan (nmap)
<!-- Fill in: command used, result, whether Suricata was added, before/after comparison -->
- **Tool:** nmap
- **Result:**
- **Detected?**
- **Notes:**

### 2. SSH brute force (Hydra)

A weak-password account was created for testing (`labuser` / `sunshine1`, isolated lab network only), and a short wordlist of common passwords was used:

```
password
123456
letmein
qwerty
sunshine1
admin
welcome1
```

```
┌──(user㉿kali)-[~]
└─$ hydra -l labuser -P pw.txt ssh://192.168.56.11 -t 4
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak
Hydra starting at 2026-11-08 14:10:07
[DATA] max 4 tasks per 1 server, overall 4 tasks, 7 login tries (l:1/p:7), ~2 tries per task
[DATA] attacking ssh://192.168.56.11:22/
[22][ssh] host: 192.168.56.11   login: labuser   password: sunshine1
1 of 1 target successfully completed, 1 valid password found
Hydra finished at 2026-11-08 14:10:14
```

**Result:** 183 total alerts generated, including **1 alert at level 12+**, 26 authentication failures, and 12 authentication successes. Top alert groups: `pam`, `authentication_failed`, `authentication_success`.

| Rule ID | Description | Level | MITRE ATT&CK |
|---|---|---|---|
| <!--fill in--> | <!--fill in--> | 12+ | T1110 (Brute Force) |

**After SSH brute force:**
<img width="1919" height="954" alt="image" src="https://github.com/user-attachments/assets/f7b0dc7e-8523-4d02-9dda-ebf1068ed408" />

<!-- Optional: note whether a longer/rockyou-style wordlist was tried and how alert volume/severity changed -->

### 3. Web reconnaissance (Nikto)

```
┌──(user㉿kali)-[~]
└─$ nikto -h http://192.168.56.11
- Nikto v2.6.1
+ Target IP:          192.168.56.11
+ Target Port:        80
+ Server: Apache/2.4.66 (Ubuntu)
+ [600050] Apache/2.4.66 appears to be outdated (current is at least 2.4.68).
+ [013587] /: Suggested security headers missing (CSP, X-Content-Type-Options,
  Referrer-Policy, Permissions-Policy, Strict-Transport-Security)
+ [007342] /: X-Frame-Options header deprecated in favor of CSP frame-ancestors
+ [007352] /: X-Content-Type-Options header not set
+ 8107 requests: 0 errors and 9 items reported on the remote host
+ End Time: 2026-11-08 14:16:23 (GMT-5) (16 seconds)
```

**Result:** 7,930 total alerts generated, including **4 alerts at level 12+**, 26 authentication failures, 14 authentication successes. Top alert groups: `web`, `accesslog`, `attack`, `recon`, `web_scan`. Top alerts included "Web server 404 error," "Multiple web server 4xx," **"XSS (Cross Site Script)"**, "Common web attack," and "Host-based anomaly."

| Rule ID | Description | Level | MITRE ATT&CK |
|---|---|---|---|
| <!--fill in--> | <!--fill in--> | 12+ | T1595 (Active Scanning) |
| <!--fill in--> | <!--fill in--> | 12+ | <!--fill in--> |
| <!--fill in--> | <!--fill in--> | 12+ | <!--fill in--> |
| <!--fill in--> | <!--fill in--> | 12+ | <!--fill in--> |

<!-- Worth a sentence here: Nikto doesn't exploit XSS, only probes for it — explain why the XSS rule fired (signature match on probe pattern vs. true positive) -->

**After web scanning attack:**
<img width="1920" height="953" alt="image" src="https://github.com/user-attachments/assets/45f90711-6235-4187-9bba-b041d2122b9e" />

---

## Detection Summary

| Attack | Tool | Alerts Generated | Level 12+ Alerts | Detected? | MITRE ID |
|---|---|---|---|---|---|
| Port scan | nmap | | | | T1046 |
| SSH brute force | Hydra | 183 | 1 | Yes | T1110 |
| Web recon/scan | Nikto | 7,930 | 4 | Yes | T1595 |

---

## Custom Detection Rules
<!-- Fill in once written: rule XML, what gap it addresses, before/after test results -->

---

## Lessons Learned
<!-- Fill in: e.g. host-based vs. network-based detection gap found with nmap, false positive investigation on XSS alert, alert volume tuning, etc. -->
