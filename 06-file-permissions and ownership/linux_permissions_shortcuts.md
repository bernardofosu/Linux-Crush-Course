# 📝 Linux File Permissions & Shortcuts

## 🔹 Meaning of Permissions

**r (Read)**\
- **File 📄** → allows reading the file content.\
Example: `cat notes.txt`\
- **Folder 📂** → allows listing contents.\
Example: `ls myfolder/`

**w (Write)**\
- **File 📄** → allows modifying the file.\
Example: editing with `nano notes.txt`.\
- **Folder 📂** → allows adding/removing files inside the directory.\
Example: `touch new.txt` or `rm old.txt` inside the folder.

**x (Execute)**\
- **File 📄** → allows executing the file as a program/script.\
Example: `./script.sh`\
- **Folder 📂** → allows entering the folder (`cd`).\
Example: `cd myfolder/`

------------------------------------------------------------------------

## 🔹 Difference between r, w, x

  Permission   On File 📄                     On Folder 📂
  ------------ ------------------------------ ----------------------------
  **r**        Read contents of file          List files inside folder
  **w**        Modify file content            Add/Remove files in folder
  **x**        Execute file (binary/script)   Enter folder with `cd`

------------------------------------------------------------------------

## 🔹 Numeric Representation (Octal)

Each permission has a numeric value:\
- r = **4**\
- w = **2**\
- x = **1**

👉 Add them up to set permissions:\
- 7 = **rwx** (full access)\
- 6 = **rw-** (read + write)\
- 5 = **r-x** (read + execute)\
- 4 = **r--** (read only)

Permissions are grouped for **Owner, Group, Others**.

Example:

``` bash
chmod 754 file.txt
```

-   7 → Owner = rwx\
-   5 → Group = r-x\
-   4 → Others = r--

So **Owner can do anything, Group can read/execute, Others can only
read**.

------------------------------------------------------------------------

## 🔹 Advantages of rwx + Numeric Permissions

✅ Fine-grained control over who can access files/folders.\
✅ Improves security by restricting unwanted access.\
✅ Numeric system makes it easy and fast to assign permissions.\
✅ Prevents accidental modification/deletion of critical files.\
✅ Ensures only trusted users can execute scripts/programs.

⚡ Example quick use:

``` bash
chmod 755 script.sh
```

-   **Owner** → rwx (full control)\
-   **Group** → r-x (read + execute)\
-   **Others** → r-x (read + execute)

Commonly used for scripts you want others to run but not modify.

------------------------------------------------------------------------

## 🔹 Symbolic (rwx) vs Numeric (Octal) Permissions

✍️ **Symbolic (rwx form)**

``` bash
chmod u=rwx,g=rx,o=r file.txt
```

-   Uses **letters**: r, w, x for user (u), group (g), others (o).\
-   More descriptive but longer to type.\
-   Useful for changing just one group's permission.

🔢 **Numeric (Octal form)**

``` bash
chmod 754 file.txt
```

-   Uses **numbers** for permissions.\
-   Shorter & faster (`chmod 755` vs `chmod u=rwx,g=rx,o=rx`).\
-   Clear at a glance (`644` = owner can edit, others only read).\
-   Consistency across multiple files/servers.\
-   Automation friendly (scripts, Ansible, Dockerfiles).

🎯 **Quick Examples**

``` bash
chmod 700 secret.txt   # only owner has access (rwx)
chmod 644 notes.txt    # owner can edit, others can read
chmod 755 script.sh    # everyone can run, only owner can modify
```

👉 Symbolic is good for teaching & one-off changes.\
👉 Numeric (755, 644, etc.) is much more practical for system admins.

------------------------------------------------------------------------

## 🔹 Shortcuts in Linux (Symbolic Links)

When you run `ls -l`, the **file type indicator** can show **l →
shortcut**.

**Meaning**\
- A shortcut (symbolic link) is a small file that just **points to
another file/folder**.\
- It doesn't contain the actual data.\
- When you open/execute the shortcut, the system redirects to the
original file.

**Example**

``` bash
ln -s /home/user/original.txt myshortcut.txt
```

-   `myshortcut.txt` is the shortcut.\
-   Opening it actually accesses `/home/user/original.txt`.

**Analogy**\
- Windows → Shortcut icon on desktop.\
- Linux → Symbolic link (`l` in `ls -l`).

✅ **In short:**\
The shortcut (symbolic link) is just a pointer --- saves you from typing
full paths or duplicating files.

------------------------------------------------------------------------

👉 Do you want me to also make a **ready-to-use cheat sheet table of
common numeric permissions** (like 777, 755, 644, etc.) for you?
