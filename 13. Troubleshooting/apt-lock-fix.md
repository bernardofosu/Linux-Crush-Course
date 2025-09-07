# 🐧 Fixing `apt` Lock Error  

### ❌ Error  
```bash
workstation@Nana-Kwasi-Fosu:~$ sudo apt update
Reading package lists... Done
E: Could not get lock /var/lib/apt/lists/lock. It is held by process 967 (apt)
N: Be aware that removing the lock file is not a solution and may break your system.
E: Unable to lock directory /var/lib/apt/lists/
```

---

### ✅ Solution  

#### 1. 🔎 Check if another `apt` process is running  
```bash
ps aux | grep apt
ps -ef | grep apt
```
- If you see only the `grep` line → no active `apt` process.  
- If another process is listed (like `apt-get` or `dpkg`), wait or kill it:  
```bash
sudo kill -9 <PID>
```

---

#### 2. 🧹 Remove leftover lock files (only when no process is running)  
```bash
sudo rm -f /var/lib/apt/lists/lock
sudo rm -f /var/cache/apt/archives/lock
sudo rm -f /var/lib/dpkg/lock-frontend
```

---

#### 3. 🔧 Reconfigure dpkg  
```bash
sudo dpkg --configure -a
```

---

#### 4. 🔄 Retry update  
```bash
sudo apt update
```

---

#### 5. ⬆️ (Optional) Upgrade packages  
```bash
sudo apt upgrade -y
```

Fix ssh login error using WSL
```sh
workstation@Nana-Kwasi-Fosu:~$ chmod 400 /mnt/c/Users/ofosu/Downloads/runners.pem

workstation@Nana-Kwasi-Fosu:~$ ssh -i /mnt/c/Users/ofosu/Downloads/runners.pem ubuntu@54.234.159.238
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0555 for '/mnt/c/Users/ofosu/Downloads/runners.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/mnt/c/Users/ofosu/Downloads/runners.pem": bad permissions
ubuntu@54.234.159.238: Permission denied (publickey).

workstation@Nana-Kwasi-Fosu:~$ ls -la /mnt/c/Users/ofosu/Downloads/runners.pem
```

Solution
```sh
workstation@Nana-Kwasi-Fosu:~$ cp /mnt/c/Users/ofosu/Downloads/runners.pem ~/.ssh/

workstation@Nana-Kwasi-Fosu:~$ chmod 600 ~/.ssh/runners.pem

workstation@Nana-Kwasi-Fosu:~$ ssh -i ~/.ssh/runners.pem ubuntu@54.234.159.238

Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1012-aws x86_64)
```


⚠️ **Important Tip:**  
🚫 Never delete lock files while another package process is running — always check with `ps aux | grep apt` first.  
