
# 🔐 Grant Limited Sudo Access for Specific Commands

To give a user limited sudo access to only certain commands or applications (like `apachectl` or `git`), you can configure this using the `sudoers` file. This is safer than giving full admin rights.

---

## ✅ Step-by-step: Grant Limited Sudo Access

1. **Open the sudoers file** (always use `visudo` to avoid syntax errors):

```bash
sudo visudo
```

2. **Add a rule for the user at the bottom.** Example: allow `tom` to restart Apache and run Git with sudo:

```bash
tom ALL=(ALL) NOPASSWD: /usr/sbin/apachectl, /usr/bin/git
```

### 🔍 Breakdown:

- `tom`: the username  
- `ALL=(ALL)`: allow on all hosts, run as all users  
- `NOPASSWD`: optional; lets user run without entering password  
- `/usr/sbin/apachectl`, `/usr/bin/git`: full path to allowed commands  

---

## 🔒 Tips

- Use `which` to get the full path:

```bash
which apachectl
which git
```

- You can restrict to specific subcommands (advanced) using `Cmnd_Alias` or shell scripts.
- Be careful with commands that can spawn a shell or edit config files — they can be used to escalate privileges.

---

## 🧪 Test it

Switch to the user and run:

```bash
sudo apachectl restart
sudo git status
```

They should work, but:

```bash
sudo bash
sudo reboot
```

...will be denied.

---

## ✅ Useful Examples

### 1. 🕸️ Restart Apache and Nginx:

```bash
tom ALL=(ALL) NOPASSWD: /usr/sbin/apachectl, /usr/sbin/service nginx restart
```

### 2. 🗂️ Run Git & Rsync:

```bash
tom ALL=(ALL) NOPASSWD: /usr/bin/git, /usr/bin/rsync
```

### 3. 🛠️ Run Custom Script:

Allow user to run a specific script:

```bash
tom ALL=(ALL) NOPASSWD: /usr/local/bin/deploy.sh
```

### 4. 📂 Edit a Specific File with nano:

```bash
tom ALL=(ALL) NOPASSWD: /bin/nano /etc/hosts
```

This restricts sudo to only editing `/etc/hosts` with nano.

### 5. 🔁 Restart a Specific Service:

```bash
tom ALL=(ALL) NOPASSWD: /bin/systemctl restart myapp.service
```

### 6. 🐍 Run Python Scripts:

```bash
tom ALL=(ALL) NOPASSWD: /usr/bin/python3 /opt/scripts/sync.py
```

### 7. 🔍 Run only tcpdump with no shell access:

```bash
tom ALL=(ALL) NOPASSWD: /usr/sbin/tcpdump
```

### 8. 🔒 Allow limited config edit via vim:

```bash
tom ALL=(ALL) NOPASSWD: /usr/bin/vim /etc/myapp/config.ini
```

---

## ❗ Tip: Prevent Shell Access

Avoid allowing sudo on general tools like:

- `/bin/bash`
- `/usr/bin/vim` (without file restriction)
- `/usr/bin/less`

These can be used to spawn a root shell.

---
