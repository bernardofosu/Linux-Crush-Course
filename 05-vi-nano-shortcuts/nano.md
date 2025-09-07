## 🧠 What Is `nano`?

`nano` is a simple, terminal-based text editor for Unix-like systems (Linux, macOS, WSL). It’s user-friendly and lightweight, making it a great choice for beginners who need to edit configuration files, code, or text files from the command line.

### ✅ Key Features:
- CLI-based text editor (no GUI required)
- Easy to use (menu at the bottom)
- Installed by default on most Linux distros
- Great for quick edits to config files (like `.bashrc`, `nginx.conf`, etc.)

---

## 🧰 Common Usage

```bash
nano filename.txt    # Opens or creates filename.txt
```
- If the file exists, it opens it.
- If it doesn't exist, nano will create a new one.

---

## 📝 Basic Nano Shortcuts (Cheat Sheet)

| Action                    | Shortcut                      | Notes                                     |
|---------------------------|-------------------------------|-------------------------------------------|
| 📝 Save (Write Out)       | `Ctrl + O`                    | "O" for "Output". Press Enter to confirm. |
| ❌ Exit                   | `Ctrl + X`                    | Prompts to save changes if needed.        |
| 🔁 Cut (Cut line)         | `Ctrl + K`                    | Cuts the entire line or marked region.    |
| 📋 Paste                  | `Ctrl + U`                    | Pastes what you cut.                      |
| 🔍 Search                 | `Ctrl + W`                    | Type what you want to find.               |
| ⏩ Next match             | `Ctrl + W` again              | Repeats the last search.                  |
| 🖍️ Start marking text     | `Ctrl + ^` (Ctrl + Shift + 6) | Use arrow keys to select text.            |
| 🔃 Undo                   | `Alt + U`                     | Undo last action.                         |
| 🔁 Redo                   | `Alt + E`                     | Redo the undone action.                   |
| 🔝 Go to start of file    | `Ctrl + _` → type `1`         | Or use arrow keys.                        |
| 🔚 Go to end of file      | `Ctrl + _` → type `9999`      | Or scroll down with arrows.               |

---

## 🖥️ Nano Bottom Bar Guide

Nano shows available commands at the bottom. The `^` symbol means `Ctrl`.

Example:
```
^G Get Help   ^O Write Out   ^W Where Is   ^K Cut Text   ^X Exit
```
- `^O` = `Ctrl + O` (save)
- `^X` = `Ctrl + X` (exit)
- `^K` = `Ctrl + K` (cut)

---

## 🧑‍💻 Example Workflow

1. Open a file:
```bash
nano config.txt
```
2. Make changes.
3. Press `Ctrl + O` to save → press `Enter` to confirm.
4. Press `Ctrl + X` to exit.

You're done! 🎉

