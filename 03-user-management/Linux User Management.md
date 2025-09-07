# 👤 Linux User Management — Quick Guide (with Emojis)

> How to create users with **`adduser`** and **`useradd`**, control home directories, set/expire passwords, lock/unlock accounts, and delete users (with or without their home). Commands assume you run as **root** or prefix with **`sudo`**. 🐧

---

## 1) ➕ Create a User with `adduser` (friendly, interactive)

`adduser` is a high‑level helper (Debian/Ubuntu) that **creates the home dir by default**, sets shell, and copies files from `/etc/skel`.

```bash
# Creates user, primary group, and /home/alice
adduser alice
# You will be prompted for a password and optional info
```

✅ **Home directory:** `/home/alice` (auto)

Verify:

```bash
getent passwd alice
ls -ld /home/alice
id alice
```

---

## 2) ➕ Create a User with `useradd` (lower‑level)

`useradd` is POSIX‑style and **non‑interactive**. You choose exactly what happens.

### 2a) Create user **with** home directory

```bash
# -m : make home dir
# -k : copy skeleton files from /etc/skel (default on many distros)
# -s : login shell (e.g., /bin/bash)
# -U : create a group with the same name (Debian/Ubuntu); on RHEL this is default
useradd -m -k /etc/skel -s /bin/bash -U bob
```

### 2b) Create user **without** home directory

```bash
# -M : do NOT create home dir (or omit -m on systems where -m is not default)
useradd -M -s /bin/bash -U deploy
```

🛠️ Manually create later:

```bash
mkdir -p /home/deploy
chown deploy:deploy /home/deploy
chmod 700 /home/deploy
usermod -d /home/deploy deploy   # set home path in account DB
```

Verify:

```bash
getent passwd bob deploy
ls -ld /home/bob /home/deploy
```

> ℹ️ System/service users: `useradd -r -s /usr/sbin/nologin appsvc` (no login shell, system UID range).

---

## 3) 🔑 Set / Update a User Password (as root)

```bash
# Set or change password now (interactive prompt)
passwd alice

# Set password from stdin (CI automation)
echo 'S3cretP@ss' | passwd --stdin alice   # (RHEL/Alma/Rocky)
# Debian/Ubuntu alternative:
echo 'alice:S3cretP@ss' | chpasswd
```

---

## 4) 🟡 Force Password Change at First Login

Make the account **expire immediately** so the user must choose a new password on next login.

```bash
# Method A (simple): expire password now
passwd -e alice

# Method B (equivalent): set last‑change day to 0
chage -d 0 alice
```

Check status:

```bash
chage -l alice
```

---

## 5) 🔒 Lock / 🔓 Unlock an Account (e.g., vacation)

Locking prevents password authentication (the hash is disabled). SSH key auth may still work unless you also disable it.

```bash
# Lock account (password logins disabled)
passwd -l alice     # or: usermod -L alice

# Unlock account
passwd -u alice     # or: usermod -U alice

# Optional: also disable shell while away
usermod -s /usr/sbin/nologin alice   # and restore with: usermod -s /bin/bash alice
```

---

## 6) 🗑️ Delete a User — Keep or Remove Home

```bash
# A) Remove user, KEEP home and mail spool (default)
userdel alice

# B) Remove user AND DELETE home dir & mail spool (⚠ irreversible)
userdel -r alice
```

If the user is currently logged in or has running processes, stop them first:

```bash
pkill -u alice              # terminate processes
# or on systemd systems:
loginctl terminate-user alice
```

Verify removal:

```bash
getent passwd alice || echo "alice gone"
ls -ld /home/alice || echo "/home/alice removed or preserved as chosen"
```

---

## 7) 🧭 Quick Reference (Cheat Sheet)

| Task                               | Command                                                                               |
| ---------------------------------- | ------------------------------------------------------------------------------------- |
| Create user (interactive)          | `adduser alice`                                                                       |
| Create with home (non‑interactive) | `useradd -m -s /bin/bash -U bob`                                                      |
| Create without home                | `useradd -M -s /bin/bash -U deploy`                                                   |
| Manually create home               | `mkdir -p /home/deploy && chown deploy:deploy /home/deploy && chmod 700 /home/deploy` |
| Set password now                   | `passwd alice`                                                                        |
| Force change at first login        | `passwd -e alice` *(or)* `chage -d 0 alice`                                           |
| Lock / Unlock                      | `passwd -l alice` / `passwd -u alice`                                                 |
| Delete user (keep home)            | `userdel alice`                                                                       |
| Delete user (remove home)          | `userdel -r alice`                                                                    |

---

### ✅ Tips

* Use **`id <user>`** and **`getent passwd <user>`** to inspect account details.
* New homes are typically created from **`/etc/skel`** (skeleton files like `.bashrc`).
* Keep **`/etc/login.defs`** and **`/etc/default/useradd`** in mind for site‑wide defaults.
* For SSH‑only users, set a shell like `/usr/sbin/nologin` and use key‑based auth.

Happy admin’ing! 🛠️🐧
