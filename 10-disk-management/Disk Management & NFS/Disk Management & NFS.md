# 💾 Disk Management & NFS

---

## 🟢 1. What is Storage?

* **Concept**: Saving data so it remains available even after shutdown.
* **Examples**:

  * 🧠 RAM → temporary (clears after reboot).
  * 💽 HDD/SSD → permanent storage.
* **Analogy**: Library 📚

  * Bookshelves = disks
  * Shelves = partitions
  * Labels & arrangement = filesystem

👉 In Linux, everything is under a single root `/` directory.

---

## 🔹 Storage in 3 Layers

1. **Raw Disk/Partition** → `/dev/sdb` / `/dev/sdb1`

   * Just blocks of data.
   * Like a blank notebook 📓.

2. **Filesystem** → `mkfs.ext4`

   * Organizes data (ext4, xfs, btrfs).
   * Like adding chapters & index 📖.

3. **Mount Point** → `/mnt/data`

   * Directory where filesystem is attached.
   * Like placing notebook on a shelf 🗂️.

✅ Without FS → useless notebook.
✅ Without mount → nobody can use it.

---

## 🔹 Demo Flow

```bash
lsblk                        # See new raw disk
sudo fdisk /dev/sdb          # Create partition
sudo mkfs.ext4 /dev/sdb1     # Make filesystem
sudo mkdir /mnt/data         # Mount point
sudo mount /dev/sdb1 /mnt/data

cd /mnt/data
touch file1.txt
```

✅ Filesystem = rules for organizing.
✅ Mounting = attach to directory tree.

---

## 🟢 2. How Linux Sees Storage

* Windows → C:, D:, E:
* Linux → devices in `/dev/`

  * `/dev/sda` → first disk
  * `/dev/sda1` → first partition

Demo:

```bash
ls /dev/xv*
```

---

## 🟢 3. Checking Disks

```bash
lsblk     # list disks
```

**Partition** = dividing disk (like C:, D:).

Inside `fdisk`:

* `n` → new
* `p` → primary
* `w` → write changes

Now you get `/dev/sdb1` 🎉

---

## 🔹 5. Formatting = Create Filesystem

```bash
sudo mkfs.ext4 /dev/sdb1
```

* 📂 ext4 = common, stable, journaling.
* 🗄️ xfs = enterprise, large scale.
* 🌀 btrfs = snapshots, checksums.

---

## 🔹 6. Mounting & Unmounting

```bash
sudo mkdir /mnt/data
sudo mount /dev/sdb1 /mnt/data

cd /mnt/data
touch testfile.txt
ls -l
```

Unmount:

```bash
sudo umount /mnt/data
```

🔒 Ensures cached writes are flushed → avoids corruption.

---

# 🌐 NFS (Network File System)

## 🔹 What is NFS?

* 📤 Share directories over network.
* 📥 Clients mount as if local.
* Invented by Sun Microsystems (1984).
* Analogy: Like Google Drive, but self-hosted ☁️.

---

## 🔑 Core Concepts

1. **NFS Server** → hosts folder.
2. **NFS Client** → mounts remote folder.
3. **Mounting** → attach remote share locally.
4. **/etc/exports** → defines rules.
5. **Protocol** → RPC, default port **2049**.

---

## 🔥 NFS Setup (2 Ubuntu VMs)

* VM1 (Server): `192.168.1.10`
* VM2 (Client): `192.168.1.11`

---

### 🟢 Step 1: Install NFS

**Server:**

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```

**Client:**

```bash
sudo apt update
sudo apt install nfs-common -y
```

---

### 🟢 Step 2: Create Share Dir (Server)

```bash
sudo mkdir -p /srv/nfs_share
sudo chown nobody:nogroup /srv/nfs_share
sudo chmod 777 /srv/nfs_share
```

---

### 🟢 Step 3: Configure Exports (Server)

Edit:

```bash
sudo nano /etc/exports
```

Add:

```
/srv/nfs_share 192.168.1.11(rw,sync,no_subtree_check)
```

Apply:

```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
```

---

### 🟢 Step 4: Mount Share (Client)

```bash
sudo mkdir -p /mnt/nfs_client
sudo mount 192.168.1.10:/srv/nfs_share /mnt/nfs_client

df -h
ls /mnt/nfs_client
```

---

### 🟢 Step 5: Persistent Mount

Edit `/etc/fstab`:

```
192.168.1.10:/srv/nfs_share  /mnt/nfs_client  nfs  defaults  0  0
```

---

## 🔒 Security in Production

1. Use real users/groups, not `nobody:nogroup`.
2. Restrict by subnet:

```
/srv/nfs_share 192.168.1.0/24(rw,sync,no_subtree_check)
```

3. Firewall:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 2049
```

4. Use Kerberos (NFSv4) for secure auth 🔐.

---

## ✅ Summary

* 📡 Server: `nfs-kernel-server` → export dir → config `/etc/exports` → restart service.
* 💻 Client: `nfs-common` → mount → add to `/etc/fstab`.
* 🛡️ Production: real users, subnet/firewall, Kerberos.

🎉 Storage & NFS basics done 🚀
