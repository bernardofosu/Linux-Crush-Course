# Setup Linux Environment on Windows and MacOS

There are multiple ways to setup a Linux environment on a Windows or Mac machines such as `cloud vm`, `wsl2`, `virtualbox`, `Hyperkit` e.t.c.,. However what I would recommend is using a container as a Linux environment.

Just install Docker desktop, run the below command and create linux container of any distribution without worrying about the cost and connectivity issues.

### Docker Command to Run Ubuntu Linux Container in windows host (Persistent & Long-Term) 

```bash
docker run -dit \
  --name ubuntu-container \
  --hostname ubuntu-dev \
  --restart unless-stopped \
  --cpus="2" \
  --memory="4g" \
  --mount type=bind,source=C:/ubuntu-data,target=/data \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -p 2222:22 \
  -p 8080:80 \
  --env TZ=Asia/Kolkata \
  --env LANG=en_US.UTF-8 \
  ubuntu:latest /bin/bash
                 
```

### Docker Command to Run Ubuntu Linux Container in mac or linux host (Persistent & Long-Term) 

```bash
docker run -dit \
  --name ubuntu-container \
  --hostname ubuntu-dev \
  --restart unless-stopped \
  --cpus="2" \
  --memory="4g" \
  --mount type=bind,source=/tmp/ubuntu-data,target=/data \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -p 2222:22 \
  -p 8080:80 \
  --env TZ=Asia/Kolkata \
  --env LANG=en_US.UTF-8 \
  ubuntu:latest /bin/bash
                 
```

## Explanation of Each Parameter

| Parameter | Description |
|-----------|-------------|
| `-dit` | Runs the container in **detached (-d)**, **interactive (-i)**, and **terminal (-t)** mode. |
| `--name ubuntu-container` | Assigns a name to the container for easy management. |
| `--hostname ubuntu-dev` | Sets the container’s hostname. |
| `--restart unless-stopped` | Ensures the container restarts automatically unless manually stopped. |
| `--cpus="2"` | Limits the container to **2 CPU cores**. |
| `--memory="4g"` | Allocates **4GB RAM** to the container. |
| `--mount type=bind,source=C:/ubuntu-data,target=/data` | **Mounts a folder** from Windows into the container to persist data. |
| `-v /var/run/docker.sock:/var/run/docker.sock` | Allows running Docker commands inside the container (optional). |
| `-p 2222:22` | Maps port **2222** on the host to **22** (SSH) inside the container. |
| `-p 8080:80` | Maps port **8080** on the host to **80** (for web services). |
| `--env TZ=Asia/Kolkata` | Sets the **timezone** (modify based on your location). |
| `--env LANG=en_US.UTF-8` | Sets the **language** settings inside the container. |
| `ubuntu:latest /bin/bash` | Uses the latest **Ubuntu** image and runs Bash shell. |



## ✅ Basic Command to Exec into a Running Container and run your linux commands
```sh
docker exec -it <container_name_or_id> /bin/bash
```
- -i = interactive
- -t = allocate a pseudo-TTY (for terminal-like behavior)
- /bin/bash = the shell you want to run (can also use /bin/sh)

### 🔍 Example
```sh
docker ps
```
(Lists running containers — get the container name or ID)
```sh
docker exec -it my_container_name /bin/bash
```

### 🛠️ If bash Doesn’t Work (Alpine or BusyBox images)
Some containers don’t have bash, so use:
```sh
docker exec -it my_container_name /bin/sh
```

### 🐳 To Get a Shell as Root User
```sh
docker exec -u 0 -it my_container_name /bin/bash
```
Or:
```sh
docker exec -u root -it my_container_name /bin/sh
```

# ✅ How to Install WSL2 on Windows

---

## Step 1: Enable Required Windows Features

1. Click **Start** → search:  
   🔍 `Turn Windows features on or off`
2. Enable the following by checking them:
   - ✅ **Windows Subsystem for Linux**
   - ✅ **Virtual Machine Platform**
3. Click **OK** → Let it apply → **Restart your PC** when prompted.

---

## Step 2: Install WSL

1. Open **PowerShell** or **Command Prompt** as **Administrator**
2. Run:
   ```powershell
   wsl --install
   ```
   > This installs the WSL2 components and sets WSL2 as the default version.

---

## Step 3: Open Microsoft Store

1. Search for a Linux distribution like:
   - ✅ **Ubuntu 22.04 LTS** (recommended)
2. Click **Install**

---

## Step 4: Reboot Your Computer

- Restart your system to finalize everything.

---

## Step 5: Launch the Linux Distro for First-Time Setup

1. Click **Start** → Open **Ubuntu** (or whichever distro you installed)
2. A terminal window will open and say:  
   _“Installing, this may take a few minutes…”_
3. When prompted:
   - **Enter a username** for your default Linux user
   - **Create a password**

> ✅ This is your Linux account — you can add other users later using standard Linux commands.

---

## 🔍 Step 6: Verify Installation

Open **PowerShell** (regular user is fine):

### ✅ Check WSL installation status:
```powershell
wsl --status
```

### ✅ Check installed Linux distros:
```powershell
wsl --list --verbose
```
Or:
```powershell
wsl -l -v
```

Expected output:
```sh
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

---

You're now ready to use WSL2 with a full Linux terminal environment! 🐧💻

