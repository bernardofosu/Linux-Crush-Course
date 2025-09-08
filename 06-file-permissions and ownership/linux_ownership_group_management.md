# 📝 Linux Ownership & Group Management

## 🔹 1. Change File/Folder Ownership

Use the **chown** command.

**Change owner only:**

``` bash
sudo chown nana file.txt
```

➡️ Makes **nana** the new owner of `file.txt`.

**Change owner and group:**

``` bash
sudo chown nana:devs file.txt
```

-   Owner = **nana**\
-   Group = **devs**

**Change ownership for folder + contents:**

``` bash
sudo chown -R nana:devs /var/www/
```

-   `-R` = recursive (apply inside all subfolders/files).

------------------------------------------------------------------------

## 🔹 2. Add User to a Supplementary (Secondary) Group

Use `usermod -aG` (append to groups).

``` bash
sudo usermod -aG docker nana
```

➡️ Adds **nana** to the `docker` group (without removing other groups).

Check groups:

``` bash
groups nana
```

------------------------------------------------------------------------

## 🔹 3. Change Primary Group of a User

Every user has one **primary group**.

**Change it:**

``` bash
sudo usermod -g devs nana
```

➡️ Now **devs** is the primary group of **nana**.

**Create a new group and set as primary:**

``` bash
sudo groupadd planners
sudo usermod -g planners nana
```

------------------------------------------------------------------------

## 🔹 Difference Between Primary & Supplementary Groups

-   **Primary group** → default group for new files the user creates.\
-   **Supplementary groups** → extra groups the user can access for
    permissions.

**Example:**\
- If nana's **primary group** = planners → any file he creates will
belong to **planners**.\
- If he's in **supplementary group** docker → he can also run Docker
commands.

------------------------------------------------------------------------

## ✅ In short

-   `chown` → change file ownership.\
-   `usermod -aG` → add supplementary group.\
-   `usermod -g` → change primary group.
