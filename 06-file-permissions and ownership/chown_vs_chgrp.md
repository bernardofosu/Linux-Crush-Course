# 📝 Difference Between `chgrp` and `chown -Rf :docker file`

## 🔹 chgrp

**Purpose** → change **only the group ownership** of a file/folder.

**Syntax:**

``` bash
chgrp docker file.txt
```

**Effect** → sets the group of `file.txt` to **docker**, without
touching the owner.

👉 Equivalent to:

``` bash
chown :docker file.txt
```

------------------------------------------------------------------------

## 🔹 chown -Rf :docker file

**Purpose** → `chown` changes **owner and/or group**.

-   `:docker` → keep the current owner, but set group = `docker`.\
-   `-R` = recursive (apply to all files/subdirs inside).\
-   `-f` = force (suppress error messages).

**Example:**

``` bash
chown -Rf :docker /var/run/docker.sock
```

-   Keeps the same file owner.\
-   Sets group = `docker`.\
-   Applies recursively if it's a directory.

------------------------------------------------------------------------

## 🔹 Key Differences

  ----------------------------------------------------------------------------------------------------
  Command                    What it changes   Recursive?   Example
  -------------------------- ----------------- ------------ ------------------------------------------
  `chgrp docker file`        Only group        No (unless   `chgrp docker file.txt`
                                               `-R`)        

  `chown -Rf :docker file`   Group (owner      Yes (`-R`)   `chown -Rf :docker /var/run/docker.sock`
                             unchanged)                     
  ----------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## 📝 chown vs chgrp Cheat Sheet

### 🔹 Change Owner Only

``` bash
chown nana file.txt
```

-   Owner = `nana`\
-   Group unchanged

------------------------------------------------------------------------

### 🔹 Change Group Only

``` bash
chgrp docker file.txt
```

➡️ Same as:

``` bash
chown :docker file.txt
```

-   Owner unchanged\
-   Group = `docker`

------------------------------------------------------------------------

### 🔹 Change Owner + Group

``` bash
chown nana:docker file.txt
```

-   Owner = `nana`\
-   Group = `docker`

------------------------------------------------------------------------

### 🔹 Recursive (Apply to Directory + Contents)

``` bash
chown -R nana:docker /var/www/
```

-   All files/folders inside `/var/www` → Owner = `nana`, Group =
    `docker`

``` bash
chgrp -R docker /var/www/
```

-   Only group is changed (recursively).

------------------------------------------------------------------------

### 🔹 Force (Suppress Errors)

``` bash
chown -Rf :docker /var/run/
```

-   `-f` hides errors (e.g., if file doesn't exist).\
-   Group = `docker` recursively.

------------------------------------------------------------------------

## ✅ Summary Table

  ------------------------------------------------------------------------------
  Action                Command (chown)                Command (chgrp)
  --------------------- ------------------------------ -------------------------
  Change owner only     `chown nana file.txt`          ❌ Not possible

  Change group only     `chown :docker file.txt`       `chgrp docker file.txt`

  Change owner + group  `chown nana:docker file.txt`   ❌ Not possible

  Recursive owner +     `chown -R nana:docker dir/`    ❌ (group only with `-R`)
  group                                                

  Recursive group only  `chown -R :docker dir/`        `chgrp -R docker dir/`

  Force (suppress       `chown -f nana:docker file`    `chgrp -f docker file`
  errors)                                              
  ------------------------------------------------------------------------------

------------------------------------------------------------------------

## ⚡ Rule of Thumb

-   Use **`chgrp`** when you only care about groups.\
-   Use **`chown`** when you want full flexibility (owner, group,
    recursive, force, etc.).
