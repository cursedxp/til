# Git Editors Guide - Choosing and Using Text Editors

When you run commands like `git commit` without `-m`, Git opens a text editor. This guide helps you choose and use the right editor.

---

## Quick Comparison

| Editor | Difficulty | Best For | Setup Time |
|--------|-----------|----------|------------|
| **Nano** | ⭐ Easy | Beginners | 30 seconds |
| **VSCode** | ⭐⭐ Moderate | Most developers | 1 minute |
| **Vim** | ⭐⭐⭐⭐⭐ Hard | Power users | Hours to master |
| **Emacs** | ⭐⭐⭐⭐⭐ Hard | Emacs users | Days to master |

---

## Recommended Setup by Experience Level

### Complete Beginner → Use Nano
```bash
git config --global core.editor "nano"
```

### Developer with IDE Experience → Use VSCode
```bash
git config --global core.editor "code --wait"
```

### Terminal Power User → Use Vim (if you already know it)
```bash
git config --global core.editor "vim"
```

---

## Nano - The Beginner-Friendly Editor

### Why Choose Nano?

✅ **Pros**:
- Shows all commands at the bottom
- No modes to remember
- Works like a regular text editor
- Installed by default on most systems
- Easy to learn in 5 minutes

❌ **Cons**:
- Basic features only
- Not as powerful as Vim/Emacs
- Less efficient for large files

### Setup

```bash
# Set as default Git editor
git config --global core.editor "nano"

# Test it
git commit
```

### How to Use Nano

When Git opens Nano, you'll see something like:

```
Write your commit message here

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch main
# Changes to be committed:
#       modified:   file.txt
#

^G Get Help  ^O Write Out  ^W Where Is  ^K Cut Text   ^J Justify
^X Exit      ^R Read File  ^\ Replace   ^U Paste Text ^T To Spell
```

**Basic Commands** (^ means Ctrl):

| Command | Action |
|---------|--------|
| Just type | Add text (no insert mode needed!) |
| `Ctrl+O` | Save (WriteOut) |
| `Enter` | Confirm filename |
| `Ctrl+X` | Exit |
| `Ctrl+K` | Cut line |
| `Ctrl+U` | Paste |
| `Ctrl+W` | Search |
| `Ctrl+G` | Help |

### Example: Writing a Commit Message

```
1. Type your message:
   feat: add user authentication

   Implements JWT-based authentication
   with login and signup endpoints.

2. Press Ctrl+O (save)
3. Press Enter (confirm)
4. Press Ctrl+X (exit)

Done! Git commits your changes.
```

### If You Make a Mistake

```bash
# Abort commit by saving empty file:
Ctrl+X  (exit without saving)
Y       (yes, exit)
```

---

## VSCode - The Modern Editor

### Why Choose VSCode?

✅ **Pros**:
- Familiar interface if you use VSCode
- Syntax highlighting
- Git integration built-in
- Full editing power (multicursor, etc.)
- Large ecosystem of extensions

❌ **Cons**:
- Requires VSCode installed
- Opens new window (slower than terminal editors)
- Overkill for simple commit messages

### Setup

```bash
# Install VSCode first (if not installed)
# Download from https://code.visualstudio.com

# Set as Git editor
git config --global core.editor "code --wait"

# The --wait flag is crucial!
# It tells VSCode to wait until you close the file
```

### How to Use VSCode for Git

When you run `git commit`, VSCode opens with a file like:

```
COMMIT_EDITMSG

(Type your message here)

# Please enter the commit message for your changes...
```

**Steps**:
1. Type your commit message
2. Save the file (`Cmd+S` or `Ctrl+S`)
3. Close the tab or window
4. Git continues with the commit

### Tips for VSCode

```bash
# Enable spell check
# Install "Code Spell Checker" extension

# View Git diff while writing message
# VSCode shows changes in side panel

# Use Gitmoji extension
# For emoji in commits (if your team uses them)
```

### If You Make a Mistake

```bash
# Abort commit:
# Delete all content from file
# Save and close
# OR just close without saving (if file is empty)
```

---

## Vim - The Power User's Choice

### Why Choose Vim?

✅ **Pros**:
- Extremely powerful once mastered
- Fast for those who know it
- Available on every Unix system
- Small and fast
- No mouse needed

❌ **Cons**:
- **VERY** steep learning curve
- Modal editing is confusing for beginners
- Easy to get stuck if you don't know commands
- Can be frustrating at first

### Only Use Vim If:
- You already know Vim
- You're willing to invest time learning
- You work in terminal extensively
- You like keyboard-only workflows

### Setup

```bash
# Vim is usually the default, but to be sure:
git config --global core.editor "vim"
```

### Vim Survival Guide

**The modes**: Vim has different modes where keys do different things.

#### Getting Unstuck in Vim

```
If you're stuck in Vim:
1. Press Esc (multiple times if needed)
2. Type :q!
3. Press Enter

This quits without saving.
```

### Basic Vim Commands for Git Commits

```
When Vim opens:

1. Press i           → Enter INSERT mode (now you can type)
2. Type your message → Write your commit message
3. Press Esc         → Exit INSERT mode (back to NORMAL mode)
4. Type :wq          → Write (save) and Quit
5. Press Enter       → Confirm

Done!
```

### Vim Modes

**NORMAL mode** (default):
- Keys are commands, not text
- `h j k l` = move left/down/up/right
- `dd` = delete line
- `u` = undo
- `:` = enter command mode

**INSERT mode** (activated by `i`):
- Type normally
- Press `Esc` to go back to NORMAL mode

**COMMAND mode** (activated by `:` in NORMAL mode):
- `:w` = save
- `:q` = quit
- `:wq` = save and quit
- `:q!` = quit without saving

### Common Vim Commands for Commit Messages

```bash
i          # Enter insert mode (start typing)
Esc        # Return to normal mode
:wq        # Save and quit
:q!        # Quit without saving (abort commit)

# While in normal mode:
dd         # Delete current line
u          # Undo
Ctrl+r     # Redo
yy         # Copy line
p          # Paste
```

### If You Get Stuck in Vim

```
Problem: I pressed something and now weird things are happening!
Solution: Press Esc multiple times, then type :q! and press Enter

Problem: I can't type anything!
Solution: Press i to enter insert mode

Problem: I typed :wq but it appeared in my text!
Solution: Press Esc first to exit insert mode, then type :wq

Problem: Nothing is working!
Solution: Close the terminal and try again
```

### Learning Vim Properly

If you want to actually learn Vim:

```bash
# Built-in tutorial (30 minutes)
vimtutor

# Or use interactive websites:
# - https://www.openvim.com/
# - https://vim-adventures.com/
```

---

## Other Editors

### Emacs

```bash
git config --global core.editor "emacs"
```

Similar complexity to Vim, different philosophy. Only use if you already know Emacs.

### Sublime Text

```bash
# macOS
git config --global core.editor "subl -w"

# Windows
git config --global core.editor "'c:/program files/sublime text/subl.exe' -w"

# Linux
git config --global core.editor "subl -w"
```

### Atom

```bash
git config --global core.editor "atom --wait"
```

Note: Atom is discontinued, not recommended for new users.

---

## Checking Your Current Editor

```bash
# See what editor Git is using
git config --global core.editor

# If nothing shows, Git uses default (usually Vim or nano)
```

---

## Avoiding Editors Completely

### Use -m flag for simple commits

```bash
# No editor needed!
git commit -m "fix: resolve navbar bug"

# Multi-line commit
git commit -m "feat: add user authentication" -m "Implements JWT tokens with login/signup endpoints"
```

### Use -e to force editor open

```bash
# Open editor even when using -m (for complex messages)
git commit -e
```

---

## Decision Flowchart

```
Do you know Vim already?
├─ Yes → Use Vim
└─ No
   │
   Do you use VSCode?
   ├─ Yes → Use VSCode
   └─ No
      │
      Are you a complete beginner?
      ├─ Yes → Use Nano
      └─ No → Use VSCode or Nano
```

---

## Quick Setup Commands

### Copy-paste one of these:

```bash
# For beginners - Nano
git config --global core.editor "nano"

# For VSCode users
git config --global core.editor "code --wait"

# For Vim users
git config --global core.editor "vim"

# For those who want to avoid editors
# (Just use -m flag always)
```

---

## Practice Exercise

Try each editor to see what you prefer:

```bash
# 1. Try Nano
git config --global core.editor "nano"
git commit --allow-empty
# Type message, Ctrl+O, Enter, Ctrl+X

# 2. Try VSCode (if installed)
git config --global core.editor "code --wait"
git commit --allow-empty
# Type message, save, close tab

# 3. Try Vim (if brave)
git config --global core.editor "vim"
git commit --allow-empty
# Press i, type message, Esc, :wq, Enter
```

---

## Troubleshooting

### Editor doesn't open
```bash
# Check if editor is installed
which nano    # or vim, code, etc.

# If not installed, install it:
# Ubuntu/Debian
sudo apt install nano

# macOS (Homebrew)
brew install nano
```

### VSCode opens but Git doesn't wait
```bash
# Add --wait flag
git config --global core.editor "code --wait"
```

### Wrong editor keeps opening
```bash
# Check system default
echo $EDITOR

# Override with Git config
git config --global core.editor "nano"
```

### Can't get out of Vim
```bash
# Press these keys in order:
Esc
:q!
Enter

# If still stuck, close the terminal window
```

---

## Recommendation for This Project

Based on your situation (learning Git basics), I recommend:

**Start with Nano** → It's the easiest and least frustrating
```bash
git config --global core.editor "nano"
```

**Later, switch to VSCode** → If you use it for development
```bash
git config --global core.editor "code --wait"
```

**Learn Vim later** → Only if you want to invest the time
```bash
# After finishing vimtutor
git config --global core.editor "vim"
```

---

## Summary

| If you want... | Use this |
|----------------|----------|
| Easiest option | Nano |
| Familiar interface | VSCode |
| No editor at all | `git commit -m "message"` |
| Terminal efficiency | Vim (after learning) |
| To avoid confusion | Nano |

**My advice**: Start with Nano, it's simple and gets the job done!

```bash
git config --global core.editor "nano"
```

---

## Navigation

- **Back to**: [Git Basics](git-basics.md)
- **See Also**: [Common Scenarios](git-scenarios.md)
- **Full Guide**: [Git Commands Guide](git-commands-guide.md)