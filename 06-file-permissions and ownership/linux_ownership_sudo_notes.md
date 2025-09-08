# 📝 Linux Ownership, sudo, and sudoers Notes (with Examples for User nana)

## 🔹 Ownership in Linux

Every file/folder has 3 levels of ownership:\
- **Owner (User)** → usually the creator.\
- **Group** → team members with shared access.\
- **Others** → everyone else.

**Example:**

``` bash
ls -l file.txt
-rw-r--r-- 1 nana devs 1024 Sep 8 13:00 file.txt
```

-   Owner = **nana** → read + write\
-   Group = **devs** → read only\
-   Others = everyone else → read only

------------------------------------------------------------------------

## 🔹 sudo Command

**sudo = SuperUser DO**\
Lets a normal user run commands as root (admin).

**Examples:**

``` bash
sudo apt update
sudo systemctl restart nginx
```

------------------------------------------------------------------------

## 🔹 sudoers File (/etc/sudoers)

Defines who can run what commands as root/admin.\
You should edit it with:

``` bash
sudo visudo
```

------------------------------------------------------------------------

## 🔹 Common sudoers Configurations

### 1. Full Access for a User

``` bash
nana ALL=(ALL) ALL
```

-   User **nana** can run all commands as root.

------------------------------------------------------------------------

### 2. Passwordless sudo

``` bash
nana ALL=(ALL) NOPASSWD: ALL
```

-   User **nana** can run all commands without entering password.\
    ⚠️ Less secure, but useful for automation.

------------------------------------------------------------------------

### 3. Group Access

``` bash
%sudo ALL=(ALL:ALL) ALL
```

-   All members of the sudo group can run any command as root.

Add **nana** to sudo group:

``` bash
sudo usermod -aG sudo nana
```

------------------------------------------------------------------------

### 4. Limit to Specific Commands

``` bash
nana ALL=(ALL) /bin/systemctl restart nginx, /usr/bin/apt-get
```

-   **nana** can only restart Nginx and run apt-get.\
-   They cannot run other root commands.

------------------------------------------------------------------------

### 5. Allow Running as Specific User

``` bash
nana ALL=(www-data) ALL
```

-   **nana** can run commands as **www-data** user.

Example:

``` bash
sudo -u www-data whoami
# Output: www-data
```

------------------------------------------------------------------------

## 🔹 Advantages of sudo System

✅ Safer than logging in as root.\
✅ Logs show who ran which command.\
✅ Granular control (specific commands/users).\
✅ Flexible (can allow/deny password prompts).

------------------------------------------------------------------------

## ✅ In short

-   **Ownership** (nana, group, others) decides file access.\
-   **sudo** lets nana act as root/admin.\
-   **sudoers** defines exact privileges: full access, passwordless,
    group-based, or limited commands.

------------------------------------------------------------------------

# 📝 sudoers Configurations Cheat Sheet

  -------------------------------------------------------------------------------------------------
  Configuration                                   Meaning          Example Usage
  ----------------------------------------------- ---------------- --------------------------------
  `nana ALL=(ALL) ALL`                            User **nana**    `sudo apt update`
                                                  can run any      
                                                  command as root  
                                                  (password        
                                                  required).       

  `nana ALL=(ALL) NOPASSWD: ALL`                  User **nana**    `sudo systemctl restart nginx`
                                                  can run any      (no password prompt)
                                                  command without  
                                                  a password.      

  `%sudo ALL=(ALL:ALL) ALL`                       All members of   Add user:
                                                  the sudo group   `sudo usermod -aG sudo nana`
                                                  can run any      
                                                  command as root. 

  `nana ALL=(ALL) /bin/systemctl restart nginx`   User **nana**    `sudo systemctl restart nginx`
                                                  can only restart 
                                                  nginx as root.   

  `nana ALL=(www-data) ALL`                       User **nana**    `sudo -u www-data whoami`
                                                  can run commands 
                                                  as the www-data  
                                                  user.            

  `root ALL=(ALL:ALL) ALL`                        The root user    Default in sudoers
                                                  has full         
                                                  unrestricted     
                                                  access.          
  -------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## 🔹 Notes

-   Always edit with **visudo** → it checks syntax before saving.\
-   `%group` = applies rule to a whole group.\
-   `NOPASSWD` = convenient for automation, but less secure.\
-   **Specific commands** = safer than full ALL.

------------------------------------------------------------------------

## ✅ In short

-   Use **ALL** for full access.\
-   Use **NOPASSWD** for automation.\
-   Use **%group** for teams.\
-   Use **command restrictions** for security.
