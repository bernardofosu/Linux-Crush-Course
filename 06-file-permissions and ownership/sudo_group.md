# 📝 How the sudo Group Works

## 🔹 1. The Idea of sudo

-   **sudo = SuperUser DO**\
-   Lets a normal user temporarily run commands as root.\
-   Safer because:
    -   You only use root when needed.\
    -   Everything done as root is logged in `/var/log/auth.log`.

------------------------------------------------------------------------

## 🔹 2. The sudo Group in Ubuntu/Debian

On Ubuntu/Debian, any user in the **sudo** group can use `sudo`.

Why? Because in `/etc/sudoers` there's:

    %sudo   ALL=(ALL:ALL) ALL

-   `%sudo` → all users in the **sudo** group.\
-   `ALL=(ALL:ALL) ALL` → they can run any command as any user
    (including root).

**Example:**

``` bash
sudo usermod -aG sudo nana
```

👉 Now **nana** can run any command with sudo.

------------------------------------------------------------------------

## 🔹 3. Why It Feels Like "Access to Many Things"

-   Normal groups (e.g., `devteam`) → file/folder access only.\
-   **sudo group** → system-wide privilege escalation.

With sudo group you can:\
- Install/remove software\
- Manage users/groups\
- Change file ownership\
- Restart services\
- Shutdown/reboot system

------------------------------------------------------------------------

## 🔹 4. Security Notes

-   Members of sudo group = administrators.\
-   Adding someone = giving root powers.

**Good practice:**\
- Keep group small.\
- Use strong passwords.\
- Restrict commands in `/etc/sudoers` if needed.

------------------------------------------------------------------------

## 🔹 5. Quick Example

Add user to sudo group:

``` bash
sudo usermod -aG sudo nana
```

Verify:

``` bash
groups nana
# Output: nana sudo
```

Run command:

``` bash
sudo apt update
```

👉 Works because **nana** is in the sudo group.

✅ In short:\
- Normal groups → file/folder access.\
- sudo group → full admin powers via sudo.

------------------------------------------------------------------------

# 📝 Root vs sudo vs Primary Group

## 🔹 1. Root User

-   Real **superuser**, UID=0.\
-   Has all permissions.

## 🔹 2. sudo

-   Not a user → it's a **program/command**.\

-   Lets normal users act as root (if sudoers allows).\

-   `/etc/sudoers`:

        %sudo ALL=(ALL:ALL) ALL

## 🔹 3. The sudo Group

-   Just a Linux group, but **linked to sudoers**.\
-   Users in this group can run sudo.

Example:

``` bash
sudo usermod -aG sudo nana
```

## 🔹 4. Primary Group on User Creation

When you create a new user (e.g., nana):

``` bash
sudo adduser nana
```

-   Primary group `nana` is created automatically.\
-   If you want a different one:\

``` bash
sudo useradd -g devteam nana
```

------------------------------------------------------------------------

## ✅ Putting It Together

-   **root** = ultimate superuser.\
-   **sudo** = command to act as root.\
-   **sudo group** = special group tied to sudoers.\
-   **primary group** = default group per user (usually same as their
    name).

⚡ Example:\
- You create nana → primary group = nana.\
- Add nana to sudo → now nana can act as root.\
- Add nana to docker → now nana can run Docker without sudo.

------------------------------------------------------------------------

# 📝 Limiting sudo Powers with sudoers

## 🔹 Edit sudoers Safely

``` bash
sudo visudo
```

------------------------------------------------------------------------

## 🔹 1. Allow User to Run Only Specific Command

``` bash
nana ALL=(ALL) /bin/systemctl restart nginx
```

-   ✅ Works: `sudo systemctl restart nginx`\
-   ❌ Fails: `sudo apt update`

------------------------------------------------------------------------

## 🔹 2. Multiple Allowed Commands

``` bash
nana ALL=(ALL) /bin/systemctl restart nginx, /usr/bin/apt-get update
```

------------------------------------------------------------------------

## 🔹 3. Restrict by Group

``` bash
%webadmins ALL=(ALL) /bin/systemctl restart nginx
```

------------------------------------------------------------------------

## 🔹 4. Passwordless for Specific Command

``` bash
nana ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
```

------------------------------------------------------------------------

## 🔹 5. Deny Dangerous Commands

``` bash
nana ALL=(ALL) ALL, !/usr/bin/apt-get
```

------------------------------------------------------------------------

## ✅ In short

-   `nana ALL=(ALL) ALL` → full sudo powers.\
-   `nana ALL=(ALL) /path/to/command` → specific command only.\
-   `%groupname` → assign sudo powers to group.\
-   `NOPASSWD:` → skip password prompt for listed command.\
-   `!command` → deny specific command.
