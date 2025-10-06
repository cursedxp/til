# Git Basics - Understanding Core Concepts

## What is Git?
Git is a **distributed version control system** that tracks changes in your code over time. Think of it as a time machine for your project that lets you:
- Save snapshots (commits) of your work
- Work on different features simultaneously (branches)
- Collaborate with others without conflicts
- Undo mistakes and go back in time

---

## Key Concepts

### 1. Repository (Repo)
A folder that Git is tracking. Contains all your files plus a hidden `.git` folder with the entire history.

```
my-project/
├── .git/          # Git's database (don't touch!)
├── src/           # Your code
├── README.md
└── package.json
```

---

### 2. Working Directory, Staging Area, and Repository

Git has three main areas where your files can be:

```
Working Directory  →  Staging Area  →  Local Repository  →  Remote Repository
(your files)          (git add)        (git commit)         (git push)
```

**Working Directory**: Your actual files where you make changes
**Staging Area (Index)**: Files you've marked to be included in the next commit
**Local Repository**: Committed snapshots stored in `.git/`
**Remote Repository**: Server copy (like GitHub, GitLab)

**Example workflow:**
```bash
# 1. Edit file
echo "Hello" > file.txt

# 2. Stage it (prepare for commit)
git add file.txt

# 3. Commit it (save snapshot locally)
git commit -m "Add greeting"

# 4. Push it (send to remote server)
git push
```

---

### 3. Commits
A **commit** is a snapshot of your project at a specific point in time. Each commit has:
- **SHA hash**: Unique ID (e.g., `a3f5b89`)
- **Author**: Who made the change
- **Timestamp**: When it was made
- **Message**: What changed
- **Parent**: Previous commit(s)

Think of commits as save points in a video game.

---

### 4. Branches
A **branch** is an independent line of development. The default branch is usually `main` or `master`.

```
main:     A---B---C---D
                   \
feature:            E---F
```

- `main` is the stable production code
- `feature` is where you develop new features
- Eventually `feature` gets merged back into `main`

**Why use branches?**
- Work on features without breaking main code
- Multiple people can work simultaneously
- Easy to experiment and throw away failed attempts

---

### 5. HEAD
`HEAD` is a pointer to your current position in the Git history. Usually points to the latest commit on your current branch.

```bash
HEAD         # Current commit
HEAD~1       # One commit back
HEAD~2       # Two commits back
HEAD^        # Parent commit (same as HEAD~1)
```

---

### 6. Remote
A **remote** is a version of your repository hosted on a server (GitHub, GitLab, etc.). Common remotes:
- `origin`: Default name for the main remote
- `upstream`: Often used for the original repo when you fork

```bash
git remote -v  # See your remotes
# origin  https://github.com/user/repo.git (fetch)
# origin  https://github.com/user/repo.git (push)
```

---

### 7. Merge vs Rebase

**Merge**: Combines branches by creating a new "merge commit"
```
main:     A---B---C-------F
                   \     /
feature:            D---E
```

**Rebase**: Replays your commits on top of another branch (cleaner history)
```
main:     A---B---C
                   \
feature:            D'---E'
```

Merge preserves history exactly as it happened. Rebase creates a linear history (cleaner, but rewrites history).

---

### 8. Fast-Forward
When Git can simply move the branch pointer forward without creating a merge commit.

```
Before:
main:     A---B
               \
feature:        C---D

After merge:
main:     A---B---C---D
```

---

### 9. Detached HEAD State
When HEAD points directly to a commit instead of a branch. Happens when you checkout a specific commit.

```bash
git checkout a3f5b89  # Now in detached HEAD state
# To fix: git checkout main
```

---

### 10. Git Editors

Git uses a text editor for commit messages when you don't use `-m`:

**Vim** (default on many systems):
- Hard to use for beginners
- Commands: `i` (insert), `Esc` (exit insert), `:wq` (save & quit), `:q!` (quit without saving)

**Nano** (beginner-friendly):
- Shows commands at bottom
- `Ctrl+O` to save, `Ctrl+X` to exit

**VSCode** (recommended):
- Familiar interface
- Set it: `git config --global core.editor "code --wait"`

---

## Quick Start Guide

### First Time Setup
```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor (optional)
git config --global core.editor "nano"

# View your settings
git config --list
```

### Creating a Repository
```bash
# Option 1: Start from scratch
mkdir my-project
cd my-project
git init

# Option 2: Clone existing repository
git clone https://github.com/user/repo.git
```

### Basic Workflow
```bash
# 1. Check status
git status

# 2. Make changes to files
echo "Hello" > file.txt

# 3. Stage changes
git add file.txt
# Or stage all changes
git add .

# 4. Commit changes
git commit -m "Add greeting message"

# 5. Push to remote
git push
```

### Viewing History
```bash
# See commit history
git log

# Compact view
git log --oneline

# See what changed
git diff

# See staged changes
git diff --staged
```

---

## Navigation

- **Next**: [Common Scenarios & Solutions](git-scenarios.md)
- **See Also**: [Quick Reference](git-quick-reference.md)
- **Advanced**: [Workflows & Best Practices](git-workflows.md)