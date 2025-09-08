# 📝 Package Management in Linux

## 🔹 What is Package Management?

A **package** = software bundled with its files, metadata, version, and
dependencies.

**Package management** = installing, updating, removing, and managing
software on Linux.

------------------------------------------------------------------------

## 🔹 What is a Package Management Tool?

A package management tool:\
- Installs new packages.\
- Resolves and installs dependencies.\
- Updates existing software.\
- Removes packages cleanly.

👉 Without them → manual download, compile, configure (slow &
error-prone).

------------------------------------------------------------------------

## 🔹 Why Do We Need Package Management Tools?

✅ Automates installation → no manual compilation.\
✅ Resolves dependencies → avoids "dependency hell".\
✅ Maintains security → quick updates/patches.\
✅ Standardizes software management → easier for admins.\
✅ Saves time → one command installs everything needed.

------------------------------------------------------------------------

## 🔹 Debian/Ubuntu Package Management

Debian-based systems use **.deb** packages.

**Low-level tool → dpkg**\
- Installs or removes .deb files directly.\
- ❌ Does not resolve dependencies automatically.

Examples:

``` bash
sudo dpkg -i package.deb
sudo dpkg -r package
```

**High-level tool → apt (Advanced Package Tool)**\
- Works on top of dpkg.\
- Downloads from repositories.\
- ✅ Handles dependencies automatically.

Examples:

``` bash
sudo apt update
sudo apt install nginx
```

**Universal packaging → snap**\
- Canonical's containerized package format.\
- Packages include dependencies → works across distros.\
- Easier for developers but **larger size & slower**.

Example:

``` bash
sudo snap install vscode --classic
```

👉 **Which is best/fastest?**\
- `dpkg` = fastest but manual.\
- `apt` = best for daily use.\
- `snap` = flexible but heavy.

------------------------------------------------------------------------

## 🔹 Red Hat / CentOS / Fedora Package Management

Red Hat-based systems use **.rpm** packages.

**Low-level tool → rpm**\
- Installs .rpm packages directly.\
- ❌ No dependency resolution.

Examples:

``` bash
sudo rpm -ivh package.rpm
sudo rpm -e package
```

**High-level tools → yum & dnf**\
- Both manage dependencies.\
- `yum` = older tool.\
- `dnf` = modern replacement (faster, better deps).

Examples:

``` bash
sudo yum install nginx   # old systems
sudo dnf install nginx   # new systems
```

------------------------------------------------------------------------

## 🔹 Low-level vs High-level Package Managers

  ---------------------------------------------------------------------------
  Type         Examples         What it does            Limitations
  ------------ ---------------- ----------------------- ---------------------
  Low-level    dpkg (Debian),   Install/remove packages ❌ No auto dependency
               rpm (RedHat)     directly                handling

  High-level   apt, yum, dnf,   Fetch packages from     Slightly slower, rely
               zypper           repos, auto resolve     on low-level tools
                                dependencies            underneath
  ---------------------------------------------------------------------------

------------------------------------------------------------------------

## ✅ Summary

-   Debian/Ubuntu → `.deb` → **dpkg (low-level), apt (high-level), snap
    (universal)**.\
-   RedHat/Fedora → `.rpm` → **rpm (low-level), yum/dnf (high-level)**.

**Best daily use:**\
- Ubuntu/Debian → `apt`\
- RHEL/Fedora → `dnf`

⚡ In short:\
- **Low-level (dpkg, rpm)** → raw installs, manual deps.\
- **High-level (apt, yum, dnf)** → smarter, auto deps, repos.\
- **Snap/Flatpak** → universal, but heavier.

👉 Do you want me to also prepare a **visual comparison table** (Debian
vs RedHat family tools)?

------------------------------------------------------------------------

# 📝 Difference Between `apt remove` vs `apt purge`

🔹 `sudo apt remove nginx`\
- Removes program binaries.\
- Keeps **config files** in `/etc/nginx/`.\
- Reinstall later → old config still there.

🔹 `sudo apt purge nginx`\
- Removes **everything**: binaries + configs + service settings.\
- Reinstall later → fresh default configs.

✅ **In short:**\
- `remove` = uninstall software, keep configs.\
- `purge` = uninstall software + delete configs.

------------------------------------------------------------------------

# 📝 Difference Between `apt update` vs `apt upgrade`

🔹 `sudo apt update`\
- Updates **package index** (list of available versions).\
- ❌ Doesn't install anything.\
- Like checking the restaurant menu.

🔹 `sudo apt upgrade`\
- Actually **upgrades installed packages** using updated list.\
- ✅ Installs new versions but never removes packages.\
- Like ordering food from the updated menu.

✅ **In short:**\
- `apt update` = refresh list of available packages.\
- `apt upgrade` = upgrade currently installed software.

⚡ Analogy:\
- `apt update` = check what's new in the app store.\
- `apt upgrade` = actually install the updates.


# 📝 Installing & Using Docker on Ubuntu (with Permissions)

## 🔹 1. Why Add Official Repository?

The default **apt repository** often has **older versions** of software
(e.g., Docker).\
To get the **latest stable release**, we add Docker's official repo.

**Steps (example for Docker):**

``` bash
# Update system
sudo apt update
sudo apt install ca-certificates curl gnupg

# Add Docker’s official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package index
sudo apt update
```

------------------------------------------------------------------------

## 🔹 2. Install Docker

``` bash
sudo apt install docker-ce docker-ce-cli containerd.io
```

-   `docker-ce` = Docker Community Edition.\
-   After install, only **root user** can run Docker.

------------------------------------------------------------------------

## 🔹 3. Docker Group & Permissions

When Docker is installed, a system group **docker** is created.\
Any user in this group can run Docker without `sudo`.

Add current user to docker group:

``` bash
sudo usermod -aG docker $USER
```

------------------------------------------------------------------------

## 🔹 4. Apply Group Membership (important!)

Changes won't apply until you refresh your session. Options:

Run:

``` bash
newgrp docker
```

➡️ Activates the new group immediately in current shell.

Or restart the shell:

``` bash
exec /bin/bash
```

Or simply logout and log back in.

------------------------------------------------------------------------

## 🔹 5. Test Docker Without sudo

``` bash
docker run hello-world
```

If it works **without sudo**, permissions are set correctly.

------------------------------------------------------------------------

## ✅ Summary

-   Add official repo → ensures **latest version**.\
-   Install Docker via apt.\
-   Add user to docker group (created automatically).\
-   Run `newgrp docker` / restart shell / logout-login.\
-   Verify with `docker run hello-world`.

------------------------------------------------------------------------

## ⚡ In short:

-   apt repo → outdated, so add official repo.\
-   root-only after install, so join docker group.\
-   group change requires refresh (`newgrp` / `exec` / relogin).



# 📝 Installing Prometheus via tar.gz (Manual Method)

📌 Official download page → https://prometheus.io/download/

------------------------------------------------------------------------

## 🔹 1. Download Prometheus

You can use either **wget** or **curl**:

``` bash
# Using wget
wget https://github.com/prometheus/prometheus/releases/download/v3.6.0-rc.0/prometheus-3.6.0-rc.0.linux-amd64.tar.gz

# Or using curl
curl -LO https://github.com/prometheus/prometheus/releases/download/v3.6.0-rc.0/prometheus-3.6.0-rc.0.linux-amd64.tar.gz
```

------------------------------------------------------------------------

## 🔹 2. Extract the tar.gz File

``` bash
tar -xvf prometheus-3.6.0-rc.0.linux-amd64.tar.gz
```

-   **x** → extract\
-   **v** → verbose (show files being extracted)\
-   **f** → file

------------------------------------------------------------------------

## 🔹 3. Navigate Into the Folder

``` bash
cd prometheus-3.6.0-rc.0.linux-amd64
ls
```

You'll see files like:\
- **prometheus** → main binary\
- **promtool** → config checker\
- **prometheus.yml** → default config file

------------------------------------------------------------------------

## 🔹 4. Run Prometheus

``` bash
./prometheus --config.file=prometheus.yml
```

Starts Prometheus using the default config.\
By default, it runs on **port 9090**.

------------------------------------------------------------------------

## 🔹 5. Access Prometheus UI

Open browser →

    http://<your-server-ip>:9090

Example: `http://13.232.56.17:9090`

You'll see the **Prometheus query interface**.

------------------------------------------------------------------------

## ✅ Summary

-   `wget / curl` → download.\
-   `tar -xvf` → extract.\
-   Run binary (`./prometheus`) → start Prometheus.\
-   Access UI on `:9090`.

⚡ This method is **fast for testing**, but for **production** you'd:\
- Move the binaries to `/usr/local/bin/`.\
- Set up a **systemd service** to run Prometheus in background.

------------------------------------------------------------------------

# ⚙️ Run Prometheus as a systemd service (Ubuntu/Debian)

Assumes you already downloaded & extracted the tarball and you're inside
the folder containing `prometheus` & `promtool`.

------------------------------------------------------------------------

## 1) Create a dedicated user & directories

``` bash
# system user without login
sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus

# config + data dirs
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
```

------------------------------------------------------------------------

## 2) Move binaries into PATH

``` bash
sudo mv prometheus promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
```

------------------------------------------------------------------------

## 3) Provide a minimal config

``` bash
sudo tee /etc/prometheus/prometheus.yml >/dev/null <<'EOF'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
EOF

sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

------------------------------------------------------------------------

## 4) Create the systemd unit

``` bash
sudo tee /etc/systemd/system/prometheus.service >/dev/null <<'EOF'
[Unit]
Description=Prometheus Time Series Database
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.listen-address=0.0.0.0:9090 \
  --storage.tsdb.retention.time=15d \
  --web.enable-lifecycle
Restart=on-failure
RestartSec=5

# Hardening (optional but recommended)
NoNewPrivileges=true
ProtectSystem=full
ProtectHome=true
PrivateTmp=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
CapabilityBoundingSet=
LockPersonality=true
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target
EOF
```

------------------------------------------------------------------------

## 5) Start and enable

``` bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

------------------------------------------------------------------------

## 6) Verify

``` bash
systemctl status prometheus
journalctl -u prometheus -f          # live logs
ss -lntp | grep 9090                 # port listening
```

Now open: `http://<server-ip>:9090`

------------------------------------------------------------------------

## 🔧 Common tweaks

**Firewall (UFW):**

``` bash
sudo ufw allow 9090/tcp
```

**Change retention:**\
Edit `--storage.tsdb.retention.time=30d` in the unit, then:

``` bash
sudo systemctl daemon-reload
sudo systemctl restart prometheus
```

**Validate config changes:**

``` bash
promtool check config /etc/prometheus/prometheus.yml
```

**Hot-reload config (lifecycle enabled):**

``` bash
curl -X POST http://localhost:9090/-/reload
```

------------------------------------------------------------------------

## 🔄 Updating Prometheus later

Stop service:

``` bash
sudo systemctl stop prometheus
```

Download new tarball, extract, then:

``` bash
sudo mv prometheus promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}
```

Start:

``` bash
sudo systemctl start prometheus
```

------------------------------------------------------------------------
