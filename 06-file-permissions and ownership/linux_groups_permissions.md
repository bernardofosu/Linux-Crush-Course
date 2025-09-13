# 📝 Linux Groups & Permissions

## 🔹 1. Create a New Group

``` bash
sudo groupadd devteam
```

Creates a new group called **devteam**.

Check if group exists:

``` bash
getent group devteam
```

------------------------------------------------------------------------

## 🔹 2. Add a User to a Group

**Add user to group (supplementary/secondary):**

``` bash
sudo usermod -aG devteam nana
```

-   `-aG` → append to group (don't remove existing groups).

**Change user's primary group:**

``` bash
sudo usermod -g devteam nana
```

**Check groups of a user:**

``` bash
groups nana
```

------------------------------------------------------------------------

## 🔹 3. Create a Folder for the Group

``` bash
sudo mkdir /project
sudo chown :devteam /project
```

-   `:devteam` → only group changes to `devteam`.\
-   Owner stays the same.

Now the group **devteam** owns `/project`.

------------------------------------------------------------------------

## 🔹 4. Set Group Permissions

``` bash
sudo chmod 770 /project
```

-   `770` →
    -   Owner: rwx\
    -   Group: rwx\
    -   Others: no access

So only the owner and group **devteam** can read/write/execute inside
`/project`.

------------------------------------------------------------------------

## 🔹 5. Set Default Group for New Files (SGID)

To make sure all new files/folders created inside `/project`
automatically belong to **devteam**:

``` bash
sudo chmod g+s /project
```

Check:

``` bash
ls -ld /project
```

➡️ You'll see `drwxrws---` (the `s` means SGID is set).

------------------------------------------------------------------------

## 🔹 6. (Optional) Give Special Permissions

**Set Sticky Bit (for shared folders):**

``` bash
sudo chmod +t /project
```

Ensures users can only delete their own files, even if group has write
access.

Example: `/tmp` folder has sticky bit set → multiple users write there
but can't delete each other's files.

------------------------------------------------------------------------

## ✅ Example Workflow

**Create group:**

``` bash
sudo groupadd devteam
```

**Add users:**

``` bash
sudo usermod -aG devteam nana
sudo usermod -aG devteam aditya
```

**Create shared folder:**

``` bash
sudo mkdir /project
sudo chown :devteam /project
sudo chmod 770 /project
sudo chmod g+s /project
```

Now both **nana** and **aditya** (group members) can collaborate in
`/project`.

------------------------------------------------------------------------

# 📝 chown Owner and Group Rules

## 🔹 Syntax

``` bash
sudo chown owner:group file_or_dir
```

-   **Before the colon (:)** → sets the owner (user).\
-   **After the colon (:)** → sets the group.

## 🔹 Cases

**Change both owner and group:**

``` bash
sudo chown nana:devteam /project
```

-   Owner = `nana`\
-   Group = `devteam`

**Change only owner:**

``` bash
sudo chown nana /project
```

-   Owner = `nana`\
-   Group stays the same.

**Change only group:**

``` bash
sudo chown :devteam /project
```

-   Owner stays the same.\
-   Group = `devteam`.

------------------------------------------------------------------------

## ✅ In short:

-   `sudo chown owner:group file` → change both.\
-   `sudo chown owner file` → change only owner.\
-   `sudo chown :group file` → change only group.

------------------------------------------------------------------------

# 📝 Using Linux Groups for Shared Access

## 🔹 Steps

1.  **Create a Group**\

``` bash
sudo groupadd devteam
```

2.  **Assign Group Ownership**\

``` bash
sudo mkdir /project
sudo chown :devteam /project
```

3.  **Give Group Permissions**\

``` bash
sudo chmod 770 /project
```

4.  **Add Users to the Group**\

``` bash
sudo usermod -aG devteam nana
sudo usermod -aG devteam aditya
```

5.  **Refresh Group Membership**\

``` bash
newgrp devteam
```

(or log out and log back in).

6.  **Test Access**\

``` bash
su - nana
cd /project
touch test.txt   # works because nana is in devteam
```

7.  **Set SGID (optional)**\

``` bash
sudo chmod g+s /project
```

➡️ Ensures all new files belong to group **devteam**.

------------------------------------------------------------------------

## ✅ In short:

-   Create group.\
-   Assign folder to group.\
-   Give group permissions.\
-   Add users to group.\
-   Users get access automatically.
