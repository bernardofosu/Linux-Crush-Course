# 👤 Create a Linux User and Install SSH Keys (RSA vs ED25519)

> Goal: create a user named **`linux`**, generate your own SSH keys locally, and install the **public** key on the VM so you can log in password‑lessly. Includes differences between **RSA** and **ED25519**, what the **fingerprint hash** means, and what to do when the user has no `~/.ssh` yet. 🐧🔐

---

## 🧭 TL;DR

1. **Generate a key** on your laptop (ED25519 recommended).
2. **Create user** `linux` on the VM.
3. Create `~linux/.ssh/authorized_keys` and paste your **public** key.
4. Fix **permissions**. Test login: `ssh -i ~/.ssh/linux_ed25519 linux@<IP>`.

> ℹ️ This walkthrough assumes you already have access to the VM with a privileged account (e.g., `ubuntu`, `ec2-user`, or `root`). If you’re locked out, use cloud/console rescue methods.

---

## 1) 🔑 Generate SSH keys on your laptop

### ✅ Recommended: ED25519 (fast, small, modern)

```bash
ssh-keygen -t ed25519 -a 100 -C "linux@my-laptop" -f ~/.ssh/linux_ed25519
```

### 🔁 Alternative: RSA 4096 (maximum compatibility)

```bash
ssh-keygen -t rsa -b 4096 -o -a 100 -C "linux@my-laptop" -f ~/.ssh/linux_rsa
```

**Flags explained**

* `-t` = key type (`ed25519` or `rsa`)
* `-b` = bits (RSA only; 4096 is strong)
* `-o` = modern OpenSSH key format (protects private key better)
* `-a` = KDF rounds (more = harder to brute‑force your passphrase)
* `-C` = comment/label (shown in `authorized_keys`)
* `-f` = output file prefix

Your key pair files:

* **Private** key: `~/.ssh/linux_ed25519` *(keep secret; protect with passphrase)*
* **Public** key:  `~/.ssh/linux_ed25519.pub` *(share & paste on server)*

> 📌 **Key fingerprints** (the “hash numbers” you see): a SHA256/MD5 hash of the **public key** used to identify it. View with: `ssh-keygen -lf ~/.ssh/linux_ed25519.pub`.

**ED25519 vs RSA**

* 🟢 **ED25519**: smaller keys, faster auth, modern security. Recommended if all systems support it (most do).
* 🟡 **RSA‑4096**: larger & slower, but works with some legacy/FIPS systems that disallow ED25519.

---

## 2) 👤 Create the user on the VM

Log in to the VM with a user that has `sudo` and run:

```bash
# Create user and home dir
sudo adduser linux                     # interactive
# or non‑interactive:
# sudo useradd -m -s /bin/bash linux

# (Optional) grant sudo
sudo usermod -aG sudo linux
```

> On some minimal images the user’s home is created but **no `~/.ssh` exists** — we’ll create it next.

---

## 3) 🗝️ Install the public key for user `linux`

On **your laptop**, print the **public** key:

```bash
cat ~/.ssh/linux_ed25519.pub
```

Copy the entire line. On the **VM**, as a privileged user:

```bash
# Create SSH dir & install key for the linux user
sudo -u linux mkdir -p /home/linux/.ssh
printf '%s\n' 'ssh-ed25519 AAAA... linux@my-laptop' | sudo tee -a /home/linux/.ssh/authorized_keys >/dev/null

# Correct ownership & permissions (StrictModes expects these)
sudo chown -R linux:linux /home/linux/.ssh
sudo chmod 700 /home/linux/.ssh
sudo chmod 600 /home/linux/.ssh/authorized_keys
```

If SELinux is enabled (RHEL/Alma/Rocky):

```bash
sudo restorecon -Rv /home/linux/.ssh
```

---

## 4) 🔧 SSH server settings (usually already fine)

Quick sanity check:

```bash
sudo sshd -T | grep -E 'pubkeyauthentication|passwordauthentication'
# Expect: PubkeyAuthentication yes
# Optional hardening (edit /etc/ssh/sshd_config): PasswordAuthentication no
sudo systemctl reload sshd
```

---

## 5) 🧪 Test the new login

From your laptop:

```bash
ssh -i ~/.ssh/linux_ed25519 linux@<VM_IP_or_DNS>
```

If you set a passphrase on the private key, you’ll be prompted for it (not the server password).

---

## 6) 🩺 Troubleshooting

* `Permission denied (publickey)`: check file perms (700 dir, 600 file), ownership (`linux:linux`), and that the **public** key line wasn’t wrapped.
* Verify the user exists & shell is valid: `getent passwd linux` (look for `/bin/bash` or similar).
* Inspect logs: `sudo tail -f /var/log/auth.log` (Deb/Ubuntu) or `sudo journalctl -u sshd -f` (RHEL/Fedora).
* Ensure you’re using the correct identity file: `ssh -i ~/.ssh/linux_ed25519 ...` and/or `~/.ssh/config`.

---

## 7) 📌 Notes

* This method works **only if you already have access** (console or another sudoer).
* For cloud rescues when you’re locked out, use: **AWS SSM / EC2 Instance Connect**, **Azure VMAccess/Serial Console**, **GCP metadata/OS Login**, or **attach disk to a rescue VM** and edit `authorized_keys` offline.

---

## 8) 🧾 Optional: `~/.ssh/config` convenience

```sshconfig
Host my-vm
  HostName <VM_IP_or_DNS>
  User linux
  IdentityFile ~/.ssh/linux_ed25519
  IdentitiesOnly yes
```

Now you can simply run: `ssh my-vm`.

---

🎉 You’ve created user **`linux`**, installed an SSH key, and validated login securely. Keep the **private key** safe (password‑protected + backup).
