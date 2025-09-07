# 🔐 EC2 SSH Key Recovery Playbook

A from‑scratch guide for when the **old private key is lost**. Create a new key, derive the public key locally, use **EC2 Instance Connect** (browser) to add it to `authorized_keys`, then SSH with the new key.

---

## 0) 🧭 What you’re going to do
- ✅ Create a **new EC2 key pair** in AWS (download the **private key**).
- 🖥️ On your **local machine**, derive the **public key** from that private key.
- 🌐 Use **EC2 Instance Connect** (browser) to get a temporary shell.
- ➕ Append the new **public key** to `~/.ssh/authorized_keys` on the instance.
- 🔑 SSH from local using the **new private key**.
- 🧹 *(Optional)* Remove any old keys from `authorized_keys`.

---

## 1) 🆕 Create a new key pair in AWS
**AWS Console → EC2 → Key pairs → Create key pair**  
- **Name:** `recovery-YYYYMMDD`  
- **Type:** **RSA** (widely compatible) or **ED25519** (modern)  
- **Format:** `.pem` (OpenSSH/mac/Linux; convert to `.ppk` for PuTTY on Windows)  
- **Download** the private key file (e.g., `recovery-20250906.pem`) and keep it safe.

> 🔒 You **cannot** recover the old private key. This creates a **new** one.

---

## 2) 💻 Derive the public key on your local machine
```bash
# Make sure file permissions are strict
chmod 600 ~/Downloads/recovery-20250906.pem

# Derive a public key from the private key (OpenSSH)
ssh-keygen -y -f ~/Downloads/recovery-20250906.pem > ~/Downloads/recovery-20250906.pub

# View it (you'll paste this on the server)
cat ~/Downloads/recovery-20250906.pub
```
- `chmod 600` → restricts read/write to you only (SSH refuses “too open” keys).  
- `ssh-keygen -y -f <pem>` → outputs the **public key** that matches the given private key.

---

## 3) 🔌 Connect with EC2 Instance Connect (browser “web CLI”)
**AWS Console → EC2 → Instances → Select the instance → Connect → EC2 Instance Connect (Browser-based SSH)**  
- **User name:** depends on AMI (e.g., `ubuntu` for Ubuntu, `ec2-user` for Amazon Linux).  
- Click **Connect** to open a browser terminal.

> ❗ If Instance Connect isn’t available, see **Fallbacks** below (SSM Session Manager, or detach/attach root volume).

---

## 4) ➕ Add the new public key to `authorized_keys`
In the **browser shell** (you’re now on the instance as the default user):

Create `~/.ssh` if needed and set safe perms:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Append the **public key** you created in step 2 (paste the line that starts with `ssh-ed25519` or `ssh-rsa`):
```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...YOUR_NEW_PUBKEY... comment' >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
> Replace the quoted line with the exact contents of your `recovery-20250906.pub`.

**Add the key for another user (e.g., `splunk`):**
```bash
sudo install -d -m 700 -o splunk -g splunk /home/splunk/.ssh
echo 'ssh-ed25519 AAAA...PUBKEY...' | sudo tee -a /home/splunk/.ssh/authorized_keys >/dev/null
sudo chmod 600 /home/splunk/.ssh/authorized_keys
sudo chown -R splunk:splunk /home/splunk/.ssh
```

> 📝 **What does `sudo install -d -m 700 -o splunk -g splunk /home/splunk/.ssh` do?**  
> It’s a one‑liner equivalent to `mkdir` + `chown` + `chmod`.  
> • `-d` create directory • `-m 700` perms • `-o/-g` owner/group = `splunk`.

**(Optional) Remove old/unknown keys:**
```bash
nano ~/.ssh/authorized_keys
# delete old lines, keep only the new one(s), save
```

---

## 5) 🔑 SSH from your local machine using the new private key
First ensure the instance’s **security group** allows **SSH (TCP 22)** from your IP.

```bash
ssh -i ~/Downloads/recovery-20250906.pem ubuntu@YOUR_INSTANCE_PUBLIC_IP
# or:
ssh -i ~/Downloads/recovery-20250906.pem ec2-user@YOUR_INSTANCE_PUBLIC_IP
```
- `-i <file>` → tell SSH which **private key** to use.

**WSL permission tip (“UNPROTECTED PRIVATE KEY FILE!”):** move the key into Linux home and fix perms (NTFS ignores chmod):
```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
cp ~/Downloads/recovery-20250906.pem ~/.ssh/
chmod 600 ~/.ssh/recovery-20250906.pem
ssh -i ~/.ssh/recovery-20250906.pem ubuntu@YOUR_INSTANCE_PUBLIC_IP
```

---

## 6) 🛡️ (Optional) Lock things down
- Remove any old key lines from `authorized_keys`.  
- Consider disabling password SSH:
  ```bash
  sudo nano /etc/ssh/sshd_config
  # Set/ensure:
  PasswordAuthentication no
  PubkeyAuthentication yes
  ```
  Then:
  ```bash
  sudo systemctl reload ssh
  ```

---

## 🧯 Fallbacks if EC2 Instance Connect doesn’t work
**A) AWS Systems Manager Session Manager**  
- Instance must have the **SSM agent** + an **IAM role** that allows SSM.  
- **EC2 → Instances → Connect → Session Manager** → get a shell, then edit `authorized_keys` as above.

**B) Detach/attach the root volume (last resort)**
1. **Stop** the instance.  
2. **Detach** its root EBS volume.  
3. **Attach** it to a helper instance as a secondary volume.  
4. **Mount** it, edit the relevant user’s `~/.ssh/authorized_keys` on the mounted filesystem to add your new public key.  
5. **Unmount**, detach from helper, **reattach** as the root volume to the original instance, **start** it, then SSH in with your new key.

---

## 🔁 Quick recap
- ✅ Create **new key pair** → download **private key**.  
- 🖥️ Locally: `ssh-keygen -y -f NEW.pem > NEW.pub`.  
- 🌐 Browser-SSH via **EC2 Instance Connect** → append `NEW.pub` to `~/.ssh/authorized_keys`.  
- 🔑 SSH from local using `-i NEW.pem`.  
- 🧹 Remove old keys, confirm security group, done.
