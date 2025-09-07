# 🔧 AWS EC2 Volume Rescue & SSH Recovery — Step‑by‑Step (with Emojis)

> Lost SSH access to an EC2 Linux instance? Use this **detach → mount on helper → fix → reattach** workflow to restore access safely. Includes key injection, sshd config checks, device naming rules, and quick copy/paste snippets. 🧯🐧

---

## 🛡️ Before You Start

* ✅ You have a **helper EC2** in the **same AZ**.
* 📸 **Snapshot** the broken root volume first (backup!).
* 🔑 Have a **new public key** ready (ED25519 recommended).

```bash
# Generate locally
ssh-keygen -t ed25519 -a 100 -C "rescue@laptop" -f ~/.ssh/rescue_ed25519
cat ~/.ssh/rescue_ed25519.pub   # copy this line
```

---

## 1️⃣ Detach Root Volume from Broken Instance

1. **Stop** the broken instance (AWS Console → EC2 → Instances → *Stop*).
2. EC2 → **Volumes** → select the **root** volume (usually small: 8–30 GB; device **/dev/sda1**).
3. **Detach** the volume.

> 🧠 Stopping ensures a clean filesystem; if you must hot‑detach, run `fsfreeze` first (advanced).

---

## 2️⃣ Attach to a Helper Instance

1. In **Volumes**, choose **Attach volume**.
2. Select the **helper EC2** (same AZ!).
3. Use device name **/dev/sdf** (it will appear inside Linux as **/dev/xvdf**).

---

## 3️⃣ Identify Devices on the Helper

```bash
lsblk -f
```

**Example**

```
xvda   30G  (helper’s root)
 ├─xvda1  /
 ├─xvda15 /boot/efi
 └─xvda16 /boot
xvdf   25G  (rescued root volume)
 ├─xvdf1  root filesystem (mount this)
 ├─xvdf15 EFI
 └─xvdf16 /boot
```

---

## 4️⃣ Mount the Broken Root Partition

```bash
sudo mkdir -p /mnt/recovery
sudo mount /dev/xvdf1 /mnt/recovery
ls /mnt/recovery   # should show: etc home var usr ...
```

> 🧩 If separate partitions exist (e.g., `/boot`, `/home`), mount them under `/mnt/recovery/boot`, `/mnt/recovery/home`, etc.

---

## 5️⃣ Check & Fix SSH Server (sshd)

```bash
# Main config
sudo sed -n '1,200p' /mnt/recovery/etc/ssh/sshd_config

# Drop‑in overrides
sudo sh -c 'for f in /mnt/recovery/etc/ssh/sshd_config.d/*.conf; do echo "--- $f"; sed -n "1,200p" "$f"; done 2>/dev/null'
```

✅ Ensure:

* `PubkeyAuthentication yes`
* `PasswordAuthentication no|yes` (choose your policy)
* `PermitRootLogin no` (recommended)

Enable sshd at boot (distro‑agnostic try‑all):

```bash
sudo chroot /mnt/recovery /bin/sh -c 'systemctl enable ssh || systemctl enable sshd || true'
```

---

## 6️⃣ Verify/Add Keys & Permissions

**Check existing authorized keys** (user example: `ubuntu`; adjust for `ec2-user`, `centos`, `debian`):

```bash
sudo sed -n '1,200p' /mnt/recovery/home/ubuntu/.ssh/authorized_keys 2>/dev/null || echo "no file"
```

**Add your new public key**:

```bash
PUBKEY="ssh-ed25519 AAAA... rescue@laptop"
sudo install -d -m 700 -o root -g root /mnt/recovery/home/ubuntu/.ssh
printf '%s\n' "$PUBKEY" | sudo tee -a /mnt/recovery/home/ubuntu/.ssh/authorized_keys >/dev/null
sudo chroot /mnt/recovery chown -R ubuntu:ubuntu /home/ubuntu/.ssh
sudo chmod 700 /mnt/recovery/home/ubuntu/.ssh
sudo chmod 600 /mnt/recovery/home/ubuntu/.ssh/authorized_keys
```

**RSA compatibility (optional)**:

```bash
echo 'PubkeyAcceptedKeyTypes +ssh-rsa' | sudo tee /mnt/recovery/etc/ssh/sshd_config.d/99-compat.conf
```

**SELinux (RHEL family)**:

```bash
sudo chroot /mnt/recovery restorecon -Rv /home/ubuntu/.ssh || true
```

> 🧠 Make sure the **username** matches the AMI default: `ubuntu` (Ubuntu), `ec2-user` (Amazon Linux), `centos`, `debian`.

---

## 7️⃣ Cleanly Unmount & Detach

```bash
sudo sync
sudo umount -R /mnt/recovery
```

Detach the volume from the helper (Console).

---

## 8️⃣ Reattach to the Original Instance

* Attach back as **/dev/sda1** (root). Inside Linux it becomes **/dev/xvda**.
* **Start** the instance.

---

## 9️⃣ Test SSH 🔑

From your laptop/workstation:

```bash
# Try original key (if you still have it)
ssh -i ~/.ssh/original-key.pem ubuntu@<PUBLIC_IP>

# Try the NEW rescue key
ssh -i ~/.ssh/rescue_ed25519 ubuntu@<PUBLIC_IP>
```

If you set `PasswordAuthentication yes`, you can also test password login (then turn it back off once fixed).

---

## 🧭 Device Naming Rules (Cheat Sheet)

* **Root volume on original instance**: attach as **/dev/sda1** → appears as **/dev/xvda** inside Linux.
* **Secondary attachment on helper**: attach as **/dev/sdf** → appears as **/dev/xvdf** inside Linux.
* After repair: **detach** from helper and **reattach** to original as **/dev/sda1** again.

> 🔎 NVMe‑based instances use `/dev/nvme…` names; use `lsblk -f` to confirm.

---

## 🔍 Troubleshooting & Gotchas

* 🔒 **Wrong user path**: Use the correct default user (`ubuntu`, `ec2-user`, `centos`, `debian`).
* 👮 **Cloud‑init overwrites**: Check `/mnt/recovery/var/lib/cloud/` if user‑data or key import was disabled; you can also use **User Data** to inject a key on next boot:

  ```yaml
  #cloud-config
  users:
    - name: ubuntu
      ssh_authorized_keys:
        - ssh-ed25519 AAAA... rescue@laptop
  ```
* 🔐 **LUKS‑encrypted root**: You’ll need the passphrase; otherwise rescue‑mount won’t help.
* 🔥 **Host firewalls**: On boot, ensure `sshd` listens and security groups allow **TCP 22**.
* 🧰 **Session Manager**: If the instance had SSM Agent + IAM role, you can skip disk attach and use **SSM** to add keys from a shell.

---

## ✅ Quick Copy/Paste — Append Key for a User

```bash
USER=ubuntu
PUBKEY="ssh-ed25519 AAAA... rescue@laptop"
sudo install -d -m 700 -o "$USER" -g "$USER" /mnt/recovery/home/$USER/.ssh
printf '%s\n' "$PUBKEY" | sudo tee -a /mnt/recovery/home/$USER/.ssh/authorized_keys >/dev/null
sudo chroot /mnt/recovery chown -R "$USER":"$USER" /home/$USER/.ssh
sudo chmod 700 /mnt/recovery/home/$USER/.ssh
sudo chmod 600 /mnt/recovery/home/$USER/.ssh/authorized_keys
```

---

### 🎉 You’re done!

You’ve safely rescued the volume, restored SSH access, and captured the essential rules for device naming and key injection. Keep the **snapshot** until you’ve verified everything works. Backup your **private key** (passphrase‑protected) to avoid this next time. 🗝️💾
