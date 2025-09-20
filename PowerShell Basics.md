# ⚡ PowerShell Basics — Quick Notes

Keep these handy while working in Windows PowerShell / PowerShell 7.

---

## 🧭 Change directory to Desktop

```powershell
# Works even if Desktop is redirected to OneDrive
cd ([Environment]::GetFolderPath('Desktop'))

cd C:\Users\ofosu\OneDrive\Desktop

# Alternative (explicit OneDrive path if KFB is enabled)
cd "$env:OneDrive\Desktop"
```

## 📌 Check current working directory

```powershell
pwd            # alias of Get-Location
Get-Location
```

## 🔎 List files (normal / long / hidden)

```powershell
ls                      # alias of Get-ChildItem
ls -Force               # include hidden & system files

# "Long list" style: show key columns
Get-ChildItem | Select-Object Mode, LastWriteTime, Length, Name | Format-Table -AutoSize

# Detailed view for a single item
Get-Item .\file.txt | Format-List *
```

## 🏗️ Create directories

```powershell
mkdir ProjectA                  # alias of New-Item -ItemType Directory
New-Item -ItemType Directory -Path .\Logs
```

## 📄 Create files (empty or with content)

```powershell
# Empty file
New-Item -ItemType File -Path .\notes.txt

# Create/overwrite with text
Set-Content .\notes.txt "Hello world"

# Append a line
Add-Content .\notes.txt "Another line"

# Create a file from a here-string (multi-line)
@"
First line
Second line
"@ | Set-Content .\report.txt
```

## 👀 Show hidden files only & view attributes

```powershell
ls -Hidden
(Get-Item .\notes.txt).Attributes
```

## 🔐 Permissions (ACLs)

### Quick & practical (uses `icacls`)

```powershell
# Grant current user Full Control
icacls .\notes.txt /grant %USERNAME%:F

# Grant a specific user Read & Write
icacls .\notes.txt /grant "DOMAIN\\Alice":RW

# Remove a grant
icacls .\notes.txt /remove:g "DOMAIN\\Alice"

# Show who has access
icacls .\notes.txt

# Apply changes recursively to a folder
icacls .\Logs /grant %USERNAME%:F /T
```

### Native PowerShell way (Get-Acl / Set-Acl)

```powershell
$path = '.\notes.txt'
$acl  = Get-Acl $path
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    $env:USERNAME, 'FullControl', 'Allow'
)
$acl.SetAccessRule($rule)
Set-Acl -Path $path -AclObject $acl
```

## 👑 Change ownership (requires admin)

```powershell
# Take ownership of a file
takeown /F .\notes.txt
icacls .\notes.txt /setowner %USERNAME%

# For a folder (recursive)
takeown /F .\ProjectA /R /D Y
icacls .\ProjectA /setowner %USERNAME% /T
```

## 🧰 Extra handy snippets

```powershell
# Open the real Desktop in Explorer
ii ([Environment]::GetFolderPath('Desktop'))

# Make a quick junction to OneDrive Desktop (if redirected)
New-Item -ItemType Junction -Path "$env:USERPROFILE\DesktopOD" -Target "$env:OneDrive\Desktop"
```

---

### ✅ Tip

Use `Get-Help <command> -Online` for full docs, e.g.:

```powershell
Get-Help Get-ChildItem -Online
```
