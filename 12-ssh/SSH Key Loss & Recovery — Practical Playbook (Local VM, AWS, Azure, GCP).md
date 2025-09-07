# 🔐 SSH Key Loss & Recovery — Practical Playbook (Local VM, AWS, Azure, GCP)

> Lost your **private key**? Don’t panic. This guide shows **safe work‑arounds** for local VMs and major clouds, how to inject a new public key, and how to avoid this situation next time. 🧯

---

## 🧭 TL;DR

* **Always keep backups** of private keys (password‑protected) and maintain a **break‑glass** access path (console/SSM/serial).
* You *don’t* need the old private key to recover access: you can **attach the disk** elsewhere and add a **new public key** to `authorized_keys`, or use each cloud’s **console access tools**.

---

## 🗝️ SSH Key Basics (60s)

* Your **public key** (🔓 *lock*) goes on the server in `~/.ssh/authorized_keys`.
* Your **private key** (🔑 *key*) stays with you.
* If you **lose** the private key, the server still trusts the **public key**, but you can’t prove ownership → you need to **install a new public key**.

---

## 🖥️ Local VM (KVM/VirtualBox/VMware) — No Cloud

**Goal:** Mount the VM disk on a *repair* host, add a new public key to the target user, fix perms, reattach.

### 1) 📤 Detach & Attach the Disk to a Repair VM/Host

1. **Power off** the broken VM.
2. **Attach** its virtual disk to another Linux VM/host (as `/dev/sdX` or similar).

### 2) 🔧 Mount & Inject Key

```bash
# Find partitions
lsblk -f

# Mount the root FS (example uses /dev/sdb1)
sudo mkdir -p /mnt/repair
sudo mount /dev/sdb1 /mnt/repair

# If separate /home: mount it too
# sudo mount /dev/sdb2 /mnt/repair/home

# Add your NEW public key for the target user (e.g., ubuntu)
PUBKEY="ssh-ed25519 AAAA... your@laptop"
sudo mkdir -p /mnt/repair/home/ubuntu/.ssh
echo "$PUBKEY" | sudo tee -a /mnt/repair/home/ubuntu/.ssh/authorized_keys >/dev/null
sudo chown -R 1000:1000 /mnt/repair/home/ubuntu/.ssh   # adjust UID/GID for that user
sudo chmod 700 /mnt/repair/home/ubuntu/.ssh
sudo chmod 600 /mnt/repair/home/ubuntu/.ssh/authorized_keys

# SELinux systems (RHEL/CentOS):
# sudo chroot /mnt/repair restorecon -R /home/ubuntu/.ssh || true

# Unmount & detach
sudo umount -R /mnt/repair
```

> 💡 If the system uses **LUKS** full‑disk encryption and you don’t have the passphrase, disk attach won’t help.

### 3) 🔁 Reattach Disk & Boot the Original VM

SSH with your **new** private key.

**Alternative:** If you have **console** access (VM GUI), log in as **root or another sudoer**, and append the key directly.

---

## ☁️ AWS EC2 — Options When Private Key is Lost

### ✅ Option A: **Session Manager (SSM)** (best)

**Prereqs:**

* Instance has **SSM Agent** installed (default on many AMIs).
* Attached **IAM role** with `AmazonSSMManagedInstanceCore`.
* Network allows access to SSM endpoints (public or via VPC endpoints).

**Steps:**

1. In the AWS Console → **Systems Manager → Session Manager** → Start session (or CLI).
2. You get a root/ssm-user shell. Add your new key:

```bash
sudo -u ubuntu mkdir -p ~ubuntu/.ssh
printf '%s\n' "ssh-ed25519 AAAA... your@laptop" | sudo tee -a ~ubuntu/.ssh/authorized_keys
sudo chown -R ubuntu:ubuntu ~ubuntu/.ssh && sudo chmod 700 ~ubuntu/.ssh && sudo chmod 600 ~ubuntu/.ssh/authorized_keys
```

### ✅ Option B: **EC2 Instance Connect** / EIC Endpoint

* Works without your original key by pushing a **one‑time public key** from the console/CLI.
* Ensure `sshd` allows `PubkeyAuthentication` and instance supports EIC. For private subnets, create an **EIC Endpoint**.

### ✅ Option C: **Detach EBS → Attach to Rescue Instance**

1. Stop the instance (or detach data volume carefully).
2. **Detach** the root EBS volume, **attach** it to a rescue EC2 as a secondary disk.
3. Mount and append your public key like in *Local VM* section, target path usually `/home/ec2-user/.ssh/authorized_keys` (Amazon Linux) or `/home/ubuntu/...` (Ubuntu).
4. **Reattach** to the original instance and boot.

### ✅ Option D: **User Data Injection (at next boot)**

* Temporarily set **EC2 User Data** to add your key at boot:

```bash
#cloud-config
users:
  - name: ubuntu
    ssh_authorized_keys:
      - ssh-ed25519 AAAA... your@laptop
```

* Reboot; cloud-init will write the key. Revert user-data after.

### 🟨 Option E: **EC2 Serial Console** (Nitro)

* Enable in account, allow IAM action `ec2-instance-connect:SendSerialConsoleSSHPublicKey`.
* Use to fix `authorized_keys` or `sshd_config` even when networking is broken.

---

## ☁️ Microsoft Azure — Work‑arounds

### ✅ Option A: **Reset SSH Key / Password (VMAccess Extension)**

```bash
# New SSH public key for user 'azureuser'
az vm user update \
  --resource-group <RG> --name <VM> \
  --username azureuser \
  --ssh-key-value "ssh-ed25519 AAAA... your@laptop"
```

* Also supports password reset. Requires Azure agent.

### ✅ Option B: **Azure Serial Console**

* Enable **boot diagnostics**. Use serial console to log in (requires valid creds or `root` access depending on distro).

### ✅ Option C: **Attach OS Disk to a Repair VM**

* Stop VM → **Detach OS disk** → Attach to repair VM → mount and edit `authorized_keys` → reattach.

---

## ☁️ Google Cloud (GCP) — Work‑arounds

### ✅ Option A: **Inject SSH Key via Metadata**

```bash
# Instance metadata (user must exist on VM)
gcloud compute instances add-metadata <VM> --zone <ZONE> \
  --metadata "ssh-keys=alice:ssh-ed25519 AAAA... your@laptop"

# Or project-wide (applies to all VMs)
gcloud compute project-info add-metadata \
  --metadata "ssh-keys=alice:ssh-ed25519 AAAA... your@laptop"
```

* OS Login environments: add key to your Google account and enable **OS Login**.

### ✅ Option B: **Serial Console**

* Enable serial port connection; use it to repair `authorized_keys` or `sshd_config`.

### ✅ Option C: **Attach Disk to Helper VM**

* Stop VM → detach its persistent disk → attach to helper → mount & inject key → reattach.

---

## 🧯 After You Regain Access — Clean Up & Harden

* 🔄 **Rotate keys** for any account that might be exposed.
* 🗑️ Remove stale keys from `authorized_keys`.
* 🔐 Protect `~/.ssh`: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`.
* 🚫 Consider disabling password logins: `PasswordAuthentication no`.
* 🪪 Keep a **break‑glass** path: SSM (AWS), Serial Console (Azure/GCP), or hypervisor console.
* 🧳 Store keys in a **password manager** or hardware key (FIDO2) and back them up.

---

## 📝 Notes & Caveats

* **Encrypted disks (LUKS)** require the passphrase on attach.
* **Cloud‑managed encryption** (EBS/KMS, Azure Disk Encryption, CMEK) is fine if your IAM allows attach.
* **SELinux** systems may need `restorecon -Rv /home/<user>/.ssh`.
* Some distros use different default users (`ec2-user`, `ubuntu`, `centos`, `debian`). Adjust paths/ownership accordingly.

---

## ✅ Quick Copy/Paste: Append New Key to a User

```bash
USER=ubuntu
PUBKEY="ssh-ed25519 AAAA... your@laptop"
sudo install -d -m 700 -o "$USER" -g "$USER" ~"$USER"/.ssh
printf '%s\n' "$PUBKEY" | sudo tee -a ~"$USER"/.ssh/authorized_keys >/dev/null
sudo chown "$USER":"$USER" ~"$USER"/.ssh/authorized_keys
sudo chmod 600 ~"$USER"/.ssh/authorized_keys
```

Stay calm, use the console/attach‑disk paths, and you’ll be back in quickly. 💪
