# 👥 Linux Group Management — Cheatsheet

> How to **check groups**, **create groups**, understand **default group powers**, **grant permissions to groups**, and **add users to multiple groups**. Commands assume `root` or `sudo`.

---

## 1) 🔎 Check a user’s groups

```bash
# Current user’s groups
id
id -nG                    # names only

# Specific user
id alice
id -nG alice

# Classic output style
groups                    # current user
groups alice

# Inspect a single group entry (members + GID)
getent group developers
```

> ℹ️ Group info lives in `/etc/group` (names & members) and `/etc/gshadow` (secure data).

---

## 2) ➕ Create / rename / delete a group

```bash
# Create a group
groupadd developers

# Create with a specific GID
groupadd -g 1205 ops

# Rename a group
groupmod -n eng developers

# Delete a group (won’t touch files on disk)
groupdel eng
```

---

## 3) 👤 Add / remove users to/from groups (single or multiple)

```bash
# Add a user to one group
usermod -aG developers alice     # -a = append, -G = supplementary groups

# Add a user to multiple groups
usermod -aG developers,ops,video alice

# Alternative (works on most distros)
gpasswd -a alice developers

gpasswd -d alice developers      # remove from a group
```

> 🔁 Membership applies to **new sessions**. Have the user re‑login or run `newgrp developers` to adopt the group immediately for the current shell.

---

## 4) 🧾 What permissions do groups have by default?

* Groups themselves don’t **own** permissions; **files/directories** do.
* Each file has **user / group / other** permission bits (r=4, w=2, x=1).
* Access is granted when a user is the **owner** or a member of the file’s **group owner**, or via **ACLs**.
* Some **special groups** on Linux distros map to extra capabilities through policy (sudoers, device access, etc.).

### Common Ubuntu/Debian groups (typical effects)

| Group               | Typical privilege                                                 |
| ------------------- | ----------------------------------------------------------------- |
| `sudo`              | Run commands as root via `sudo` (defined in `/etc/sudoers`)       |
| `adm`               | Read many logs in `/var/log`                                      |
| `systemd-journal`   | Read the system journal (journald)                                |
| `docker`            | Manage Docker via `/var/run/docker.sock` → **root‑equivalent** ⚠️ |
| `dialout`           | Access serial ports (`/dev/ttyS*`, `/dev/ttyUSB*`)                |
| `lpadmin`           | Administer printers                                               |
| `audio`, `video`    | Access audio/video devices                                        |
| `plugdev`, `netdev` | Hot‑plug devices / network management (desktop use)               |
| `www-data`          | Web server service user; own web files to limit write scope       |

> ⚠️ Be cautious adding users to **`sudo`** or **`docker`** — both effectively grant root.

---

## 5) 🛡️ Grant permissions *to a group* (files & directories)

### A) Classic Unix mode bits

```bash
# Make a group own a directory
chgrp -R developers /srv/project

# Give the group rwx on the directory (and r-x on existing files)
chmod -R g+rwX /srv/project

# Keep new files in this dir owned by the same group (setgid bit)
chmod 2775 /srv/project          # 2 = setgid
```

**Why setgid?** New files created under `/srv/project` will inherit the directory’s group (`developers`), ensuring collaboration.

### B) Default ACLs (fine‑grained, recursive by default)

```bash
# Grant group full access now and for all future files in the dir
setfacl -Rm g:developers:rwx /srv/project
setfacl -Rm d:g:developers:rwx /srv/project    # default ACL for new files/dirs

# View ACLs
getfacl /srv/project
```

> ACLs are great when multiple groups need different access levels.

### C) Sudo privileges (administrative commands)

```bash
# Grant admin rights by adding user to the sudo group
usermod -aG sudo alice

# Or create a limited rule (edit with visudo)
visudo
# Example line:
# %deploy  ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp
```

---

## 6) 🧪 Example: Create a shared team workspace

```bash
# 1) Create a group
groupadd developers

# 2) Add users to it
usermod -aG developers alice
usermod -aG developers bob

# 3) Create the shared directory
mkdir -p /srv/project
chgrp -R developers /srv/project
chmod 2775 /srv/project          # setgid so new files inherit the group

# 4) Ensure group has access to all existing and future files
chmod -R g+rwX /srv/project
setfacl -Rm d:g:developers:rwx /srv/project

# 5) Verify
groups alice
getent group developers
getfacl /srv/project | head
```

---

## 7) 🧭 Quick reference

| Task                    | Command                           |
| ----------------------- | --------------------------------- |
| List user’s groups      | `id -nG alice` / `groups alice`   |
| Show group entry        | `getent group developers`         |
| Create group            | `groupadd developers`             |
| Rename group            | `groupmod -n eng developers`      |
| Delete group            | `groupdel developers`             |
| Add user to many groups | `usermod -aG dev,ops,video alice` |
| Remove from a group     | `gpasswd -d alice dev`            |
| Change dir group        | `chgrp -R dev /path`              |
| Grant rwx to group      | `chmod -R g+rwX /path`            |
| Setgid on directory     | `chmod 2775 /path`                |
| Default ACL for group   | `setfacl -Rm d:g:dev:rwx /path`   |

---

### ✅ Tips

* Check `/etc/login.defs` and `/etc/adduser.conf` for site defaults (e.g., `UMASK`).
* Use **`umask 002`** in a shared group environment so new files start group‑writable.
* Encourage users to re‑login after group changes, or use `newgrp <group>` to refresh in the current shell.

Happy grouping! 🧩
