# 🔐 Linux ACL (Access Control List) – Full Practical Guide

## 🟦 1. Install ACL

### 🐧 **Ubuntu/Debian**

```bash
sudo apt install acl
```

### 🐧 **CentOS/RHEL/Fedora**

```bash
sudo yum install acl
```

### 🐧 **Arch Linux**

```bash
sudo pacman -S acl
```

---

# 🟩 2. Create Multiple Users & Groups

### ➤ Create users

```bash
sudo useradd john
sudo useradd mary
sudo useradd sam
```

### ➤ Create groups

```bash
sudo groupadd devteam
sudo groupadd qa
```

### ➤ Add users to groups

```bash
sudo usermod -aG devteam john
sudo usermod -aG qa mary
sudo usermod -aG qa sam
```

---

# 🟧 3. Create a File or Folder

```bash
sudo mkdir /project
sudo touch /project/data.txt
```

---

# 🟨 4. Add Multiple Users & Groups Using ACL

### ➤ Give **john** full access

```bash
sudo setfacl -m u:john:rwx /project/data.txt
```

### ➤ Give **mary** read-only

```bash
sudo setfacl -m u:mary:r-- /project/data.txt
```

### ➤ Give **devteam** full access

```bash
sudo setfacl -m g:devteam:rwx /project/data.txt
```

### ➤ Give **qa** read-write access

```bash
sudo setfacl -m g:qa:rw- /project/data.txt
```

---

# 🟫 5. Set Default ACL for a Folder

(Default means new files inherit permissions)

```bash
sudo setfacl -m d:u:john:rwx /project
sudo setfacl -m d:g:devteam:rwx /project
sudo setfacl -m d:g:qa:rw- /project
```

---

# 🟪 6. Check ACL Permissions

Run:

```bash
getfacl /project/data.txt
```

You will see something like:

```
user::rw-
user:john:rwx
user:mary:r--
group::r--
group:devteam:rwx
group:qa:rw-
mask::rwx
other::---
```

This confirms **multiple users and groups** have access.

---

# 🟥 7. Test Access (Switch Users)

### ➤ Test as **john** (should have rwx)

```bash
su - john
cd /project
echo "hello" >> data.txt
```

### ➤ Test as **mary** (should have read-only)

```bash
su - mary
cat /project/data.txt
# This works

nano /project/data.txt
# ❌ Permission denied
```

### ➤ Test as **sam** (qa group - should have read/write)

```bash
su - sam
cd /project
echo "qa update" >> data.txt
```

---

# 🟦 8. Final Summary

- ✔ ACL allows **multiple users** AND **multiple groups** on one file.
- ✔ ACL does NOT replace the file's main group.
- ✔ ACL permissions override standard chmod limitations.
- ✔ Default ACL makes shared folders for teams easy.

Let me know if you want examples with **folders**, **setgid**, or **removing ACL entries**.
