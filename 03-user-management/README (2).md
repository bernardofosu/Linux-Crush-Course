# 🛠 Ubuntu User Management Script

## 📌 Introduction
This Bash script allows administrators to manage user accounts on an Ubuntu system efficiently. It provides options to:

- Create a new user with an expiration date ⏳
- Delete a user ❌
- List all users 📜
- Lock a user 🔒
- Unlock a user 🔓
- Check user expiration details 🕒
- Exit the script 🚪

---

## 📝 Script Breakdown & Explanation

### 1️⃣ Checking Root Permissions
```bash
check_root() {
    if [[ "$EUID" -ne 0 ]]; then
        echo "Error: This script must be run as root." >&2
        exit 1
    fi
}
```
- `[[ "$EUID" -ne 0 ]]` → Checks if the script is being run as root (`EUID` is 0 for root users).
- `echo "Error: This script must be run as root." >&2` → Displays an error message if not run as root.
- `exit 1` → Exits the script to prevent unauthorized execution.

### 2️⃣ Creating a New User
```bash
create_user() {
    read -p "Enter username to create: " username
```
- `read -p "Enter username to create: " username` → Prompts for a username.

```bash
    if id "$username" &>/dev/null; then
        echo "User $username already exists."
        return
    fi
```
- `id "$username" &>/dev/null` → Checks if the user already exists.
- `return` → Stops execution if the user exists.

```bash
    read -s -p "Enter password for $username: " password
    echo
```
- `read -s -p "Enter password for $username: " password` → Prompts for a password without displaying it on the screen.

```bash
    read -p "Enter user expiration date (YYYY-MM-DD): " expire_date
```
- `read -p "Enter user expiration date (YYYY-MM-DD): " expire_date` → Prompts for an expiration date.

```bash
    if ! date -d "$expire_date" &>/dev/null; then
        echo "Invalid date format. User not created."
        return
    fi
```
- `date -d "$expire_date"` → Validates the entered date format.
- `return` → Stops execution if the date is invalid.

```bash
    useradd -m -s /bin/bash -e "$expire_date" "$username"
```
- `useradd -m` → Creates a new user and a home directory.
- `-s /bin/bash` → Assigns Bash shell as the default shell.
- `-e "$expire_date"` → Sets the expiration date for the user.

```bash
    echo "$username:$password" | chpasswd
```
- `echo "$username:$password" | chpasswd` → Sets the user password.

```bash
    echo "User $username created successfully. Expiration Date: $expire_date"
```
- Displays success message.

#### 🏷 Adding User to a Group
```bash
    read -p "Add user to a group? (y/n): " add_group
    if [[ "$add_group" == "y" ]]; then
        read -p "Enter group name: " groupname
        if grep -q "$groupname" /etc/group; then
            usermod -aG "$groupname" "$username"
            echo "User $username added to group $groupname."
        else
            echo "Group $groupname does not exist."
        fi
    fi
}
```
- `usermod -aG "$groupname" "$username"` → Adds user to the specified group.

---

### 3️⃣ Deleting a User
```bash
delete_user() {
    read -p "Enter username to delete: " username
```
- Prompts for the username to delete.

```bash
    if ! id "$username" &>/dev/null; then
        echo "User $username does not exist."
        return
    fi
```
- Checks if the user exists; if not, it stops execution.

```bash
    read -p "Are you sure you want to delete user '$username'? (y/n): " confirm
    if [[ "$confirm" == "y" ]]; then
        userdel -r "$username"
        echo "User $username deleted successfully."
    else
        echo "User deletion aborted."
    fi
}
```
- `userdel -r "$username"` → Deletes the user and their home directory.

---

### 4️⃣ Listing All Users
```bash
list_users() {
    echo "Listing all system users:"
    awk -F':' '{ print $1 }' /etc/passwd
}
```
- Uses `awk` to extract and print all usernames from `/etc/passwd`.

---

### 5️⃣ Locking & Unlocking Users
```bash
lock_user() {
    read -p "Enter username to lock: " username
    if id "$username" &>/dev/null; then
        passwd -l "$username"
        echo "User $username has been locked."
    else
        echo "User $username does not exist."
    fi
}
```
- `passwd -l "$username"` → Locks the user account.

```bash
unlock_user() {
    read -p "Enter username to unlock: " username
    if id "$username" &>/dev/null; then
        passwd -u "$username"
        echo "User $username has been unlocked."
    else
        echo "User $username does not exist."
    fi
}
```
- `passwd -u "$username"` → Unlocks the user account.

---

### 6️⃣ Checking User Expiration Date
```bash
check_expiration() {
    read -p "Enter username to check expiration: " username
    if ! id "$username" &>/dev/null; then
        echo "User $username does not exist."
        return
    fi
    chage -l "$username"
}
```
- `chage -l "$username"` → Displays password and account expiration details.

---

### 7️⃣ Displaying the Menu
```bash
show_menu() {
    echo "----------------------------------"
    echo "  Ubuntu User Management Script  "
    echo "----------------------------------"
    echo "1) Create a new user"
    echo "2) Delete a user"
    echo "3) List all users"
    echo "4) Lock a user"
    echo "5) Unlock a user"
    echo "6) Check user expiration"
    echo "7) Exit"
    echo "----------------------------------"
}
```
---

### 🚀 Running the Script
```bash
chmod +x user_management.sh
sudo ./user_management.sh
```

✅ **Benefits**
- Enforces security with user expiration ⏳
- Prevents inactive users from staying in the system 🔒
- Helps system administrators manage users efficiently 🛠

Hope this helps! 😊🎉

