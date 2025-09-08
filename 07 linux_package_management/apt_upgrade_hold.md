# 📝 Package Upgrade & apt-mark hold

## 🔹 apt upgrade vs apt full-upgrade

**Normal Upgrade**

``` bash
sudo apt upgrade
```

-   Upgrades installed packages to the newest version.\
-   ❌ Does **not** remove or install new packages if dependencies
    change.\
-   ✅ Safe upgrade.

**Full Upgrade (or dist-upgrade)**

``` bash
sudo apt full-upgrade
```

-   Upgrades packages **and handles dependency changes**.\
-   May **remove old packages** or **install new ones** automatically.\
-   ⚠️ More aggressive.

👉 Example:

``` bash
sudo apt update
sudo apt upgrade          # normal upgrade
sudo apt full-upgrade     # allows removals/installs for dependencies
```

------------------------------------------------------------------------

## 🔹 Holding a Package (Prevent Upgrade)

Sometimes you want to **freeze a package** at its current version (to
avoid breaking changes).

**Command:**

``` bash
sudo apt-mark hold <package_name>
```

**Example:**

``` bash
sudo apt-mark hold terraform
```

➡️ Prevents Terraform from upgrading during `apt upgrade` or
`apt full-upgrade`.

To remove the hold:

``` bash
sudo apt-mark unhold terraform
```

------------------------------------------------------------------------

## 🔹 Why Use apt-mark hold?

-   Avoid breaking production apps after upgrades.\
-   Keep a stable version of tools (e.g., Terraform, Docker, Kernel).\
-   Useful when a new release has **bugs or incompatibilities**.

✅ **In short:**\
- `upgrade` → updates packages safely.\
- `full-upgrade` → updates + removes/installs as needed.\
- `apt-mark hold` → lock package at current version.\
- `apt-mark unhold` → unlock it for future upgrades.

------------------------------------------------------------------------

# 📝 Upgrade a Specific Package in Ubuntu/Debian

## 🔹 1. Update Package Index

``` bash
sudo apt update
```

## 🔹 2. Upgrade Only One Package

``` bash
sudo apt install --only-upgrade <package_name>
```

👉 Example (only Docker):

``` bash
sudo apt install --only-upgrade docker-ce
```

➡️ Upgrades only Docker if a new version is available.\
Other packages will **not** be upgraded.

## 🔹 3. Upgrade Multiple Specific Packages

``` bash
sudo apt install --only-upgrade docker-ce containerd.io
```

## 🔹 4. Check Version Before Upgrade

See current version installed:

``` bash
apt list --installed docker-ce
```

See available versions in repo:

``` bash
apt list -a docker-ce
```

## 🔹 5. Install a Specific Version (Optional)

``` bash
sudo apt install docker-ce=<version>
```

👉 Example:

``` bash
sudo apt install docker-ce=5:24.0.7-1~ubuntu.22.04~jammy
```

## 🔹 6. Hold All Other Packages

Prevent unintended upgrades of other packages (like kernel, Terraform,
etc.):

``` bash
sudo apt-mark hold terraform
sudo apt-mark hold linux-generic
```

------------------------------------------------------------------------

## ✅ In short:

-   `apt install --only-upgrade pkgname` → upgrade only that package.\
-   `apt-mark hold` → freeze critical packages you don't want upgraded.\
-   `apt list -a pkgname` → check available versions.

------------------------------------------------------------------------

👉 Do you want me to also prepare a **cheat sheet** comparing:\
- Upgrade everything (`upgrade`, `full-upgrade`)\
- Upgrade only one package (`--only-upgrade`)\
- Pin/hold versions (`apt-mark`, specific version install)?
