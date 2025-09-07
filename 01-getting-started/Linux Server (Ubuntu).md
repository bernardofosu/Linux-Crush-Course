# 🧰 Linux Server (Ubuntu) — Day‑1 Creation Checklist

> A concise, copy‑pasteable guide for spinning up and hardening a fresh Linux machine (cloud or bare‑metal). Includes SSH keys, users, firewall, updates, disks, and optional extras. 🔐🐧

---

## 0) 🧱 Prereqs / Decisions

* ☁️ **Where**: AWS EC2 / GCP / Azure / Bare metal
* 🏷️ **OS**: Ubuntu LTS (24.04/22.04) or Debian (stable)
* 🔑 **Auth**: Prefer **SSH key‑pair** (ed25519). Avoid passwords.
* 🌐 **Ports**: Decide what must be open (22, 80, 443, custom)
* 💾 **Storage**: Root size, extra data volume, snapshot policy

**Create SSH key locally**

```bash
ssh-keygen -t ed25519 -a 100 -C "my-laptop" -f ~/.ssh/id_ed25519
# Public key is ~/.ssh/id_ed25519.pub — upload it in the cloud console or use ssh-copy-id for on‑prem
```

---

## 1) 🚀 First Login

```bash
ssh -i ~/.ssh/id_ed25519 ubuntu@<SERVER_IP>
```

If on AWS, consider attaching an **IAM role** and an **Elastic IP**.

---

## 2) 🔒 Create Admin User & Keys

```bash
# Create a user (replace alice)
sudo adduser alice
sudo usermod -aG sudo alice

# Add your public key to the new user
sudo -u alice mkdir -p /home/alice/.ssh
sudo -u alice chmod 700 /home/alice/.ssh
cat ~/.ssh/authorized_keys | sudo tee -a /home/alice/.ssh/authorized_keys >/dev/null
sudo chown -R alice:alice /home/alice/.ssh
sudo chmod 600 /home/alice/.ssh/authorized_keys
```

> ✅ Keep a second SSH session open while changing SSH settings (safety net).

---

## 3) 🧰 System Update & Essentials

```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install vim htop curl git ufw fail2ban unattended-upgrades
sudo systemctl enable --now unattended-upgrades
```

Set timezone & NTP:

```bash
sudo timedatectl set-timezone UTC
systemctl status systemd-timesyncd --no-pager
```

---

## 4) 🧱 SSH Hardening (sshd)

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudoedit /etc/ssh/sshd_config
```

Recommended minimum changes:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
# Optional: restrict which users can SSH
# AllowUsers alice
```

Apply & verify:

```bash
sudo systemctl reload sshd
sshd -t && echo OK   # config syntax check
```

Optional 2FA (later): `libpam-google-authenticator` or hardware keys (U2F/FIDO2).

---

## 5) 🔥 Firewall (UFW) + Cloud Rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH         # port 22
# If web server:
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status
```

> ☁️ Also set **Security Groups / Network Rules** in your cloud provider.

---

## 6) 💽 Disks & Mounts (if you attached a data volume)

Format & mount (example: /dev/sdb → /data):

```bash
sudo lsblk
sudo parted /dev/sdb --script mklabel gpt mkpart primary ext4 0% 100%
sudo mkfs.ext4 -L data1 /dev/sdb1
sudo mkdir -p /data
sudo mount /dev/sdb1 /data

# Persist across reboots (get the UUID)
blkid | grep sdb1
sudoedit /etc/fstab
# Add a line like:
# UUID=<the-uuid-here>  /data  ext4  defaults,nofail  0 2
```

Backups: enable **snapshots** (EBS/PDisks) and schedule them 📆.

---

## 7) 🧪 Basic Monitoring & Protection

* 🧱 **fail2ban** enabled (default sshd jail). Check: `sudo systemctl status fail2ban`.
* 📈 `htop`, `iostat`, `vmstat`, `df -h` for quick health checks.
* 🧾 Persistent logs (default `journald`). Consider shipping to CloudWatch/Stackdriver/ELK.

---

## 8) 🐳 Optional: Containers & Apps

Install Docker (rootless or standard):

```bash
# Standard (quick):
sudo apt -y install docker.io docker-compose-plugin
sudo usermod -aG docker alice  # ⚠️ docker group is root‑equivalent

# Or set up rootless Docker (more secure):
curl -fsSL https://get.docker.com/rootless | sh
```

NGINX + TLS with certbot (quick web host):

```bash
sudo apt -y install nginx certbot python3-certbot-nginx
sudo ufw allow 'Nginx Full'
# Point your DNS A record to the server, then:
sudo certbot --nginx -d example.com -d www.example.com
```

---

## 9) 🛡️ Golden Image & Automation (Nice‑to‑Have)

* 🌱 **cloud‑init** user‑data for bootstrapping (users, packages, ssh keys)
* 🤖 **Ansible** for repeatable config
* 🧱 **Packer** to bake AMIs/images; **Terraform** for infra as code

**cloud‑init sample (user‑data):**

```yaml
#cloud-config
users:
  - name: alice
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: [sudo]
    shell: /bin/bash
    ssh_authorized_keys:
      - <your-public-key>
packages:
  - htop
  - ufw
runcmd:
  - apt-get update && apt-get -y upgrade
  - ufw allow OpenSSH && ufw --force enable
```

---

## 🔑 SSH Keys Refresher (Lock & Key Analogy)

* 🔓 **Public key** = **lock** you put on the server
* 🔐 **Private key** = **key** you keep safe on your laptop
* ✅ Server verifies you own the private key — no password sent over the network

Copy your key to an existing server (non‑cloud):

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server
```

---

## ✅ Final Sanity Checks

```bash
# Login as your new user
ssh alice@<SERVER_IP>
# Confirm sudo works
sudo -v
# Confirm firewall rules
sudo ufw status numbered
# Confirm time sync
timedatectl
```

> You now have a clean, minimal, and secure Linux machine ready for production or labs. 🎉
