# 📝 Bash Backup Script & File Copying Notes

## 🔹 Bash Backup Script Breakdown

``` bash
#!/bin/bash
SOURCE="/home/ubuntu/source"
DESTINATION="/home/ubuntu/destination/"
DATE=$(date +%Y-%m-%d_%H-%M-%S)

mkdir -p $DESTINATION/$DATE
cp -r $SOURCE $DESTINATION/$DATE
echo "Backup completed on $DATE"
```

Tells the system to run the script with the Bash shell.

Without it, the computer may not know which language to use.

------------------------------------------------------------------------

### Variables

``` bash
SOURCE="/home/ubuntu/source"
DESTINATION="/home/ubuntu/destination/"
```

Variables for folders.

-   **SOURCE** = the folder to back up.\
-   **DESTINATION** = where the backup will be stored.

Analogy: "Photos are here → copy them to there."

------------------------------------------------------------------------

### Date Command

``` bash
DATE=$(date +%Y-%m-%d_%H-%M-%S)
```

-   `$(...)` runs a command and saves the result.\
-   `date` adds timestamp → e.g., `2025-09-08_14-22-30`.\
-   Ensures unique folder names so backups don't overwrite.

------------------------------------------------------------------------

### Make Directory

``` bash
mkdir -p $DESTINATION/$DATE
```

-   `mkdir -p` = make folder (including parents if missing).\
-   Creates a new dated backup folder.\
-   Example: `/home/ubuntu/destination/2025-09-08_14-22-30`

------------------------------------------------------------------------

### Copy Command

``` bash
cp -r $SOURCE $DESTINATION/$DATE
```

-   `cp -r` = copy everything (files + subfolders).\
-   Copies the source into the new dated folder.

------------------------------------------------------------------------

### Echo Command

``` bash
echo "Backup completed on $DATE"
```

-   Prints a message with the backup date for confirmation.

------------------------------------------------------------------------

✅ **In short:**\
Makes a timestamped copy of a folder.\
Steps: Create new folder → Copy everything → Print success message.

⚠️ **Limitations:**\
- Copies everything every time (no incremental backups).\
- No remote storage support.\
- No consistency checks if files are being modified.

------------------------------------------------------------------------

## 🗂️ File Copying in Linux

### 📌 cp (Copy)

**What it does:**\
Copies files/folders within the same computer.

``` bash
cp -r /home/user/docs /mnt/usb/
```

**Disadvantages:**\
- ❌ Cannot copy to another computer.\
- ❌ Always copies everything (no incremental).\
- ❌ No progress/error handling.\
- ❌ Inefficient for large backups.

**Better choices:**\
- Cross-computer transfers → **scp / rsync**.\
- Incremental, efficient backups → **rsync**.

------------------------------------------------------------------------

### 📌 scp (Secure Copy)

**What it does:**\
Copies files between two computers over SSH.

``` bash
scp -r /home/user/docs user@192.168.1.10:/home/user/backup/
```

**Disadvantages:**\
- ❌ Copies the whole file every time.\
- ❌ Slower for large files than rsync.\
- ❌ No resume if transfer breaks.

**Better choices:**\
- One-time transfers → scp is fine.\
- Ongoing syncs/large data → rsync.

------------------------------------------------------------------------

### 📌 rsync (Remote Sync)

**What it does:**\
Copies files locally (like cp) or remotely (like scp) --- but smarter.

``` bash
rsync -avz /home/user/docs user@192.168.1.10:/home/user/backup/
```

**Advantages:**\
- ✅ Incremental: only copies what changed.\
- ✅ Faster + saves bandwidth.\
- ✅ Can resume interrupted transfers.\
- ✅ Can delete removed files (--delete).\
- ✅ Works locally & over network.

------------------------------------------------------------------------

## 🎯 Quick Analogy

-   **cp** = moving things inside your house.\
-   **scp** = sending things to a friend's house (but inefficient).\
-   **rsync** = sending things smartly, only new/changed items, and can
    handle interruptions.
