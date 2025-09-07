# 🔐 SSH Key Setup for User **splunk** — Zero to Login

This is a from‑scratch checklist to create a **splunk** user, generate an SSH key on your **local** machine, add the **public key** to the server, and log in as **splunk**. Includes flag explanations for `ssh-keygen`, `ssh`, etc.

---

## ✅ 0) Prerequisites
- You can already SSH into the server as **ubuntu** (or another sudo user).
- You have a terminal on your local machine (Linux/macOS/WSL or Windows PowerShell).

---

## 🗝️ 1) Generate an SSH key on your **local** machine
**Recommended (Ed25519)**

```bash
ssh-keygen -t ed25519 -C "youremail@gmail.com"
# or with a different comment label:
ssh-keygen -t ed25519 -C "splunk@your-host"
```
**What the flags mean**
- `ssh-keygen` → creates a new SSH keypair.
- `-t ed25519` → key **type** = Ed25519 (modern, fast, secure).  
  • Legacy alternative if required: `-t rsa -b 4096`.
- `-C "..."` → **comment** to label the key (e.g., email or host tag).
- Optional extras:  
  • `-f ~/.ssh/splunk_ed25519` → save to a **custom file**.  
  • `-N 'passphrase'` → set a **passphrase** non‑interactively.

Show your **public** key (safe to share):
```bash
cat ~/.ssh/id_ed25519.pub
```

> 📝 **WSL tip:** Keep keys in `~/.ssh` (Linux home), not `/mnt/c/...` to avoid permission issues.

---

## 🧑‍🚀 2) SSH into the server as **ubuntu** and create the user
From your **local** machine:
```bash
ssh -i ~/.ssh/your_ubuntu_key ubuntu@SERVER_IP
```

Create **splunk**:
```bash
sudo adduser splunk                    # interactive (creates /home/splunk)
# Optional: grant sudo
sudo usermod -aG sudo splunk
```
**Explained**
- `adduser splunk` → makes the user + home dir + default shell.
- `usermod -aG sudo splunk` → **append** user to **sudo** group.

---

## 🔑 3) Install the public key for **splunk**
Still on the **server** (logged in as ubuntu):

Create `.ssh` with correct ownership & permissions:
```bash
sudo install -d -m 700 -o splunk -g splunk /home/splunk/.ssh
```

Append your public key to `authorized_keys`:
```bash
sudo bash -c 'cat >> /home/splunk/.ssh/authorized_keys' <<'EOF'
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...PASTE_PUBLIC_KEY_HERE... comment
EOF
sudo chmod 600 /home/splunk/.ssh/authorized_keys
sudo chown -R splunk:splunk /home/splunk/.ssh
```

**Explained**
- `install -d -m 700` → make directory with **700** perms (`rwx------`).  
- `authorized_keys` must be **600** or SSH will ignore it.

💡 **One‑liner from local (optional):**
```bash
cat ~/.ssh/id_ed25519.pub | ssh ubuntu@SERVER_IP  'sudo install -d -m 700 -o splunk -g splunk /home/splunk/.ssh &&   sudo tee -a /home/splunk/.ssh/authorized_keys >/dev/null &&   sudo chown splunk:splunk /home/splunk/.ssh/authorized_keys &&   sudo chmod 600 /home/splunk/.ssh/authorized_keys'
```

---

## 🚪 4) Log out as ubuntu, log in as **splunk**
From your **local** machine:
```bash
# If your key is the default id_ed25519:
ssh splunk@SERVER_IP

# Or specify a custom key file:
ssh -i ~/.ssh/id_ed25519 splunk@SERVER_IP
```
**Useful SSH flags**
- `-i <file>` → use this **identity** (private key).  
- `-p <port>` → SSH on a **non‑default port**.  
- `-v` / `-vvv` → verbose / very verbose (debug).

---

## 🛡️ 5) (Optional) Harden SSH **after** confirming key login
```bash
sudo nano /etc/ssh/sshd_config
```
Set/confirm:
```
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
```
Reload:
```bash
sudo systemctl reload ssh
```

---

## 🧰 Troubleshooting (quick)
- **UNPROTECTED PRIVATE KEY FILE!** (WSL) → Move key to Linux home and fix perms:
  ```bash
  mkdir -p ~/.ssh && chmod 700 ~/.ssh
  mv /mnt/c/Users/<you>/Downloads/yourkey.pem ~/.ssh/
  chmod 600 ~/.ssh/yourkey.pem
  ```
- **Permission denied (publickey)** → On server:
  ```bash
  sudo chown -R splunk:splunk /home/splunk/.ssh
  sudo chmod 700 /home/splunk/.ssh
  sudo chmod 600 /home/splunk/.ssh/authorized_keys
  ```
  Ensure the key in `/home/splunk/.ssh/authorized_keys` exactly matches your local `~/.ssh/id_ed25519.pub`.
- **Multiple keys confusion** → Force identity & debug:
  ```bash
  ssh -vvv -i ~/.ssh/id_ed25519 splunk@SERVER_IP
  ```

---

## 🧾 Quick reference: `ssh-keygen` options
- `-t ed25519` → key **type** (Ed25519 recommended).  
- `-b 4096` → **bits** (use with RSA only).  
- `-C "text"` → **comment** label (e.g., `youremail@gmail.com`, `splunk@your-host`).  
- `-f /path/file` → **filename** for the keypair (default: `~/.ssh/id_ed25519`).  
- `-N 'passphrase'` → set **passphrase** non‑interactively.
