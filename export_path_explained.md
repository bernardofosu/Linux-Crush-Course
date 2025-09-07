
# 🧾 Understanding `export PATH` and System Binary Directories

The difference between the two `export PATH` commands lies in how many system binary directories you're including, which directly affects what commands are available to your shell:

## 🧾 1. `export PATH=$PATH:/bin:/sbin`
This adds only:

- `/bin` — essential user commands (like `ls`, `cp`, `rm`, `cat`)
- `/sbin` — essential system binaries (like `shutdown`, `reboot`, `ifconfig`)

### 💡 Limitation:
You won’t have access to many common utilities, like `grep`, `find`, `tr`, `which`, etc., because they usually live in `/usr/bin` or `/usr/sbin`.

## 🧾 2. `export PATH=$PATH:/bin:/sbin:/usr/bin:/usr/sbin`
This adds everything from above, plus:

- `/usr/bin` — most user commands (`grep`, `awk`, `nano`, `python`, etc.)
- `/usr/sbin` — admin tools (`service`, `iptables`, `adduser`, etc.)

### ✅ Recommended:
Use this version for full access to almost all standard system tools.

---

## 📊 Summary Table

| Directory    | Contains                    | Needed for            |
|-------------|-----------------------------|------------------------|
| `/bin`      | Core user commands          | Everyday use           |
| `/sbin`     | Core system admin commands  | Network/tools          |
| `/usr/bin`  | Most userland utilities     | `grep`, `vim`, etc.    |
| `/usr/sbin` | Non-essential admin commands| `iptables`, `sshd`, etc.|

So, unless you have a specific reason to keep the path minimal (e.g., sandboxed environment), always use the longer version:

```bash
export PATH=$PATH:/bin:/sbin:/usr/bin:/usr/sbin
```

---

## 🛠 Fixing the Issue

You said:

> so they are there but the path that is not set

I said:

> Exactly — ✅ the commands are present on the system, but ❌ they're not accessible from your shell because their directories aren't in your `$PATH`.

For example:

- You have `grep`, `which`, `tr`, etc., installed under `/usr/bin`
- But since `/usr/bin` is not in your current `$PATH`, trying to run them gives you a `command not found` error

### ✅ Solution: Add missing paths

To fix it temporarily:

```bash
export PATH=$PATH:/bin:/sbin:/usr/bin:/usr/sbin
```

To make it permanent (on most Linux systems):

```bash
echo 'export PATH=$PATH:/bin:/sbin:/usr/bin:/usr/sbin' >> ~/.bashrc
source ~/.bashrc
```



✅ **If a program is in your $PATH:**
You can run it just by typing its name, no matter what directory you're in.

**Example:**

```bash
ls
```
Works because `ls` is in `/bin`, which is part of `$PATH`.

---

❌ **If a program is NOT in your $PATH:**
You must specify its full or relative path to run it.

**Example:**  
You installed a tool to `/home/tom/tools/mytool`.

To run it:

```bash
/home/tom/tools/mytool
```

Or, if you're in the tools directory:

```bash
./mytool
```
(You must use `./` to refer to "current directory" — because `.` is not in `$PATH` by default for security.)

---

🛠️ **To "run directly", add it to $PATH:**  
Edit your shell config (like `.bashrc`, `.zshrc`, etc.) and add:

```bash
export PATH=$PATH:/home/tom/tools
```

Then you can run:

```bash
mytool
```
from anywhere in the system.

---

✅ **If Splunk is installed at `/opt/splunk/bin`, add it to your $PATH**:

**Temporary (for current session):**
```bash
export PATH=$PATH:/opt/splunk/bin
splunk start
```

**Permanent (for all sessions):**
Add this line to your shell config (e.g., `~/.bashrc`, `~/.bash_profile`, or `~/.zshrc`):

```bash
export PATH=$PATH:/opt/splunk/bin
```

Then reload it:

```bash
source ~/.bashrc
```

---

🧭 **What Each Directory Means:**

| Directory    | Purpose                                                                 | Example Commands        |
|--------------|-------------------------------------------------------------------------|--------------------------|
| `/bin`       | Essential user commands, available to all users                         | `ls`, `cp`, `mv`        |
| `/sbin`      | System administration commands, mostly for root                         | `shutdown`, `reboot`    |
| `/usr/bin`   | User-level programs, not essential for boot or recovery                 | `python`, `vim`, `gcc`  |
| `/usr/sbin`  | System services and admin tools for multi-user systems                  | `apache2`, `sshd`       |

---

👀 **To check your current $PATH:**

```bash
echo $PATH
```
