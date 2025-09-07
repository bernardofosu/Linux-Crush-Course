# 🐧 Linux Architecture (Simple & Practical) — And What Happens When You Run `cat file.txt`

> A friendly, high‑level map of Linux internals and a step‑by‑step journey of a simple command from your keyboard to the screen.

---

## 🧱 Big Picture: Layers of Linux

```mermaid
flowchart TB
  A[👤 User / Terminal] --> B[🐚 Shell (bash/zsh)]
  B --> C[📦 User Space Programs & Libraries]\ncat, ls, grep, Python, glibc, OpenSSL
  C --> D[🧠 Kernel]:::k
  D --> E[🧩 Kernel Subsystems]:::k
  E --> F[🔌 Device Drivers]:::k
  F --> G[🧊 Hardware]\nCPU • RAM • Disk • NIC

  classDef k fill:#eef,stroke:#88a,stroke-width:1.2px
```

**User space**

* 🐚 **Shell**: parses the command, creates processes, handles pipes/redirection.
* 📦 **Apps & Libraries**: `cat`, `grep`, your apps; common C library **glibc** wraps syscalls.

**Kernel space** (monolithic, but modular):

* 🗂️ **VFS** (Virtual Filesystem), ext4/xfs/etc drivers
* 🧮 **Scheduler** (process/threads), **MM** (memory manager & page cache)
* 🌐 **Networking stack**
* 🔐 **Security** (permissions, namespaces, cgroups, SELinux/AppArmor)
* 🔌 **Drivers** (disk, GPU, NIC, TTY/PTY)

---

## 🐱 The Journey of `cat file.txt`

### 0) You type the command

* 🧑‍💻 Keystrokes go to your **terminal emulator** (e.g., GNOME Terminal)
* Terminal talks to the kernel through a **PTY** (pseudo‑terminal) pair: master ↔ slave.
* The **shell** (bash) is attached to the PTY *slave* and receives your text.

### 1) Shell starts `cat`

* Bash parses `cat file.txt`.
* 🧬 Bash uses `fork()` → creates a **child** process.
* 👟 In the child, bash calls `execve("/usr/bin/cat", ["cat","file.txt"], env)` to replace itself with `cat`.
* 🧩 ELF loader + dynamic linker (`ld-linux`) map `cat` and needed shared libraries into memory.

### 2) `cat` opens the file

* In user space, `cat` calls `open()` via **glibc** → kernel syscall **`openat()`**.
* The **kernel VFS**:

  * resolves the path (dentry/inode lookup, follows symlinks)
  * checks **permissions/ownership** and **LSM** (SELinux/AppArmor)
  * returns a **file descriptor** (FD), e.g., `3`.

### 3) Kernel reads data (fast path: page cache)

* `cat` calls **`read(fd, buf, size)`**.
* Kernel tries to serve from the **page cache** 🧠. If **cache hit** → copy to user buffer.
* If **cache miss** → kernel asks the **filesystem driver** (ext4/xfs) → **block layer & I/O scheduler** → **disk driver** → storage (NVMe/SATA). Data DMA’ed into RAM → page cache → copied to user buffer.

### 4) `cat` writes to your screen

* `cat` calls **`write(1, buf, n)`** (FD **1 = stdout**).
* FD 1 points to the PTY *slave*.
* Kernel delivers bytes to PTY *master*; terminal emulator reads and **renders** the text.

### 5) Process exit & shell prompt

* `cat` reaches EOF → calls `close()` → exits with status 0.
* Bash (parent) receives **SIGCHLD**, does `waitpid()` → prints the next prompt.

---

## 🧪 See it live with `strace`

*(Outputs vary by distro; this is typical.)*

```bash
# Trace only file I/O syscalls for cat
strace -e trace=openat,read,write,close cat file.txt
```

Sample (annotated):

```text
openat(AT_FDCWD, "file.txt", O_RDONLY)   = 3     # 🔓 get FD 3
read(3, "Hello\nWorld\n", 8192)              = 12    # 📥 kernel → user
write(1, "Hello\nWorld\n", 12)                = 12    # 📤 to PTY/terminal
close(3)                                      = 0     # 🔒 done
```

---

## 🧠 Extra: What if the file is missing or forbidden?

* ❌ Not found → kernel returns **`-ENOENT`**; shell shows `cat: file.txt: No such file or directory`.
* 🔐 No permission → kernel returns **`-EACCES`** (or `-EPERM` with LSM); shell shows `Permission denied`.

---

## 🔧 Pipes & Redirection (bonus)

* `cat file.txt | grep foo` → the shell creates a **pipe** (a kernel buffer). `write()` from `cat` feeds `read()` of `grep`.
* `cat file.txt > out.txt` → shell opens `out.txt` for writing and **replaces FD 1** (stdout) for the `cat` process before `execve()`.

---

## 🧭 TL;DR

* User types → **PTY** → **shell** → `fork+exec` → **syscalls** (`open/read/write/close`) → **kernel** (VFS, page cache, drivers) → **hardware** → bytes flow back → **terminal**.
* Linux’s **monolithic kernel** coordinates permissions, memory, scheduling, I/O, and drivers to make a tiny command feel instant ⚡.
