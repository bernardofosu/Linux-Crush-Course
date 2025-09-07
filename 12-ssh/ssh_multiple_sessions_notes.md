# 🔐 SSH Sessions & Multiple Logins – Notes

## 📊 Why multiple SSH processes appear
When you run:
```bash
pgrep ssh
```
You may see several **PIDs** at once. Each PID represents a process.

Typical reasons:
- **Master sshd daemon** → background service that accepts connections.  
- **Active SSH sessions** → each login creates its own child process.  
- **Helpers** → e.g., port forwarding, `ssh-agent`, EC2 Instance Connect.  

---

## 🔎 Example Breakdown from `ps`
```
UID          PID    PPID  C STIME TTY      STAT   TIME CMD
root        1273       1  0 Sep05 ?        Ss     0:00 sshd: /usr/sbin/sshd -D -o AuthorizedKeysCommand ...
root       77125    1273  0 19:16 ?        Ss     0:00 sshd: ubuntu [priv]
root       77127    1273  0 19:16 ?        Ss     0:00 sshd: ubuntu [priv]
ubuntu     77241   77125  0 19:16 ?        S      0:03 sshd: ubuntu@pts/0
ubuntu     77532   77127  0 19:16 ?        S      0:00 sshd: ubuntu@notty
```

- **`root 1273`** → main sshd daemon, always running.  
- **`root 77125 & 77127`** → privileged parent processes handling authentication.  
- **`ubuntu 77241`** → interactive shell session at `pts/0`.  
- **`ubuntu 77532`** → another session (non-interactive, `notty`, or second login).  

---

## ✅ Matching with `who`
```
ubuntu   pts/0   2025-09-06 19:16 (154.161.161.208)
ubuntu   pts/1   2025-09-06 19:47 (154.161.161.208)
```
- **pts/0** → your first SSH shell.  
- **pts/1** → your second SSH shell (likely another terminal/tab).  
- Both from the same IP → it’s you, logged in twice.  

---

## ⚖️ Is multiple sessions bad?
- 🟢 **Fine (normal):** multitasking, multiple terminals, automation tools, same user/IP.  
- 🔴 **Risky:** unexpected sessions, unmonitored shells, multiple admins on production.  

So, in your case (both sessions from the same IP), it’s safe.  

---

## 🔧 How to limit SSH sessions (optional)
1. **Global (sshd_config):**
   ```
   MaxSessions 1
   MaxStartups 1
   ```
   Then restart SSH:
   ```bash
   sudo systemctl restart sshd
   ```

2. **Per-user (limits.conf):**
   ```
   ubuntu hard maxlogins 1
   ```
   Edit in `/etc/security/limits.conf`.  

---

## ❓ Does group membership cause this?
No ❌ — adding `ubuntu` to `www-data` (or any group) only changes **file access permissions**.  
It does **not** create multiple SSH logins.  

Multiple sessions come from you logging in multiple times, not from group membership changes.  

---

## 🔎 Useful commands to monitor
- Show who’s logged in:
  ```bash
  who
  ```
- Show what sessions are doing:
  ```bash
  w
  ```
- Show last logins (with IP):
  ```bash
  last -a | grep ubuntu
  ```

---

## ⚡ Quick Recap
- Multiple PIDs = master daemon + child processes for each login.  
- Having 2 sessions from the same IP is safe (you just logged in twice).  
- Groups like `www-data` only affect permissions, not sessions.  
- You can limit to 1 session if needed for security.  
