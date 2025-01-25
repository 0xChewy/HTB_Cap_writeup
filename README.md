**HTB Cap Write-Up**

**Author:** 0xChewy

---

### Introduction

Cap is an easy-difficulty Linux machine hosting an HTTP server for administrative purposes, including network captures. Improper access controls result in an Insecure Direct Object Reference (IDOR) vulnerability, enabling unauthorized access to another user's capture. The capture contains plaintext credentials, which can be exploited to gain an initial foothold. Privilege escalation is achieved by exploiting a misconfigured Linux capability to gain root access.

---

### Initial Reconnaissance

#### **Nmap Scan**

A basic Nmap scan revealed three open TCP ports:

- **Port 21 (FTP):** An unsecured protocol communicating in plaintext.
- **Port 22 (SSH):** Allows remote shell access with or without authentication.
- **Port 80 (HTTP):** Provides web resources accessible via browsers.


Given the presence of a web server on port 80, I accessed the machine's web interface using a browser.

---

### Navigating the HTTP Server


Upon navigating to `http://<ip_address>`, I encountered a dashboard. To identify hidden web directories, I utilized **ffuf** with the SecLists directory wordlist. This enumeration process revealed four additional endpoints, including `/data/`.


_______
### Exploitation of IDOR Vulnerability

#### **Identifying the Vulnerability**

At `/data/`, I observed a URL parameter, e.g., `/data/9`, that suggested a potential IDOR vulnerability. By replacing the parameter value with another numeric value (`/data/0`), I gained unauthorized access to another user’s dashboard, confirming the presence of IDOR.


#### **Burp Suite Intrusion**

Using **Burp Suite**, I executed an intruder attack by fuzzing the IDOR parameter with values ranging from 0 to 30. This attack revealed that only certain user IDs had packet capture (PCAP) files stored under `/data/`.


#### **Extracting Credentials**

The PCAP file from user ID `0` contained plaintext FTP credentials captured during a session. The credentials were as follows:

- **Username:** `nathan`
- **Password:** `________`

---

### Initial Access via SSH

With the extracted credentials, I successfully logged into the machine via SSH as user `nathan`:

```bash
ssh nathan@<ip_address>
```

Upon gaining shell access, I found the first flag (`user.txt`) and the LinPEAS script within Nathan's home directory. Executing LinPEAS revealed a privilege escalation vector related to Python capabilities.

______________

### Privilege Escalation

#### **Capability Misconfiguration**

LinPEAS identified that Python on the system had a misconfigured capability allowing the `CAP_SETUID` privilege. This capability permits a process to change its user ID arbitrarily, which can be leveraged to gain root access.

Reference: https://man7.org/linux/man-pages/man7/capabilities.7.html

#### **Exploitation**

A simple Python script was crafted to escalate privileges:

```python
import os
os.setuid(0)
os.system("/bin/bash")
```


# Conclusion
_______
Running the script set my UID to 0, granting root-level access. After successful elevation, navigating to the root directory revealed the final flag (`root.txt`).
