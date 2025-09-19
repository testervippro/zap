
# 🛠️ Guide: Add `zap.sh` to PATH on macOS

## 📍 Goal

Enable running ZAP from anywhere using:

```bash
zap.sh
```

Instead of typing the full path:

```bash
/Applications/ZAP.app/Contents/Java/zap.sh
```

---

## ✅ Step 1: Locate the ZAP Executable

ZAP is usually installed at:

```bash
/Applications/ZAP.app/Contents/Java/zap.sh
```

Keep this path — we’ll add it to your system’s `PATH`.

---

## ✅ Step 2: Open Your Shell Config File

### If you use the default **zsh** shell (macOS 10.15+):

```bash
nano ~/.zshrc
```

### If you use **bash** (older systems or custom shell):

```bash
nano ~/.bash_profile
```

---

## ✅ Step 3: Add ZAP to Your PATH

At the bottom of the file, add this line:

```bash
export PATH="/Applications/ZAP.app/Contents/Java:$PATH"
```

This tells your shell where to find `zap.sh`.

---

## ✅ Step 4: Save and Apply Changes

After editing the file, save and close:

* Press `Ctrl + O` to save
* Press `Enter` to confirm
* Press `Ctrl + X` to exit

Then apply the changes:

```bash
source ~/.zshrc     # Or source ~/.bash_profile
```

---

## ✅ Step 5: Test It

Now, try running:

```bash
zap.sh -version
```

You should see ZAP's version printed in the terminal.

---

## 💡 Optional: Create a Shortcut Command

If you prefer a shorter command like `zap`, you can also add this to your config file:

```bash
alias zap="zap.sh"
```

Then apply again:

```bash
source ~/.zshrc
```

Now you can launch ZAP with just:

```bash
zap
```

---

## ✅ Done!

You can now launch ZAP easily from any terminal using `zap.sh` or `zap`.

